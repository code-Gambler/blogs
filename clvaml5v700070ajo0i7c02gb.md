---
title: "Understanding the GCC Passes"
seoTitle: "Passes in the GCC compiler"
seoDescription: "How to create passes in GCC complier"
datePublished: Mon Apr 22 2024 07:18:19 GMT+0000 (Coordinated Universal Time)
cuid: clvaml5v700070ajo0i7c02gb
slug: understanding-the-gcc-passes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1713770162713/584cdc3a-0bff-4f61-9013-955fe301cd63.png
tags: gnu, gcc, gcc-compiler, spo600, afmv

---

As I was responsible for providing the diagnostic output for the Auto Function Multi Versioning for the [AFMV](https://stevenpillay.hashnode.dev/afmv-project) Project. I started by studying how other passes in the GCC compiler work.

For this, I created a simple Hello World program in C.

```c
#include <stdio.h>
int main() {
   // printf() displays the string inside quotation
   printf("Hello, World!");
   return 0;
}
```

Saved it as `helloWorld.c`, then used the GCC compiler that we built to compile it and dump all the passes, using this command `../gcc/build/bin/gcc -fdump-tree-all -fdump-rtl-all helloWorld.c`, the dumped all the passes of optimizations that were enabled in the directory.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713766885569/bcb1fe42-0b07-4265-8808-e9f1d807117c.png align="center")

These were all the passes, I shortlisted and understood the `isel` pass, the logic for which was stored in the `gimple-isel.cc` file.

Now let's understand the `isel` pass, the main logic for the pass is as follows:

```c
namespace {

const pass_data pass_data_gimple_isel =
{
  GIMPLE_PASS, /* type */
  "isel", /* name */
  OPTGROUP_VEC, /* optinfo_flags */
  TV_NONE, /* tv_id */
  PROP_cfg, /* properties_required */
  0, /* properties_provided */
  0, /* properties_destroyed */
  0, /* todo_flags_start */
  TODO_update_ssa, /* todo_flags_finish */
};

class pass_gimple_isel : public gimple_opt_pass
{
public:
  pass_gimple_isel (gcc::context *ctxt)
    : gimple_opt_pass (pass_data_gimple_isel, ctxt)
  {}

  /* opt_pass methods: */
  bool gate (function *) final override
    {
      return true;
    }

  unsigned int execute (function *fun) final override;
}; // class pass_gimple_isel


/* Iterate all gimple statements and perform pre RTL expansion
   GIMPLE massaging to improve instruction selection.  */

unsigned int
pass_gimple_isel::execute (struct function *fun)
{
  gimple_stmt_iterator gsi;
  basic_block bb;
  hash_map<tree, unsigned int> vec_cond_ssa_name_uses;
  auto_bitmap dce_ssa_names;
  bool cfg_changed = false;

  FOR_EACH_BB_FN (bb, fun)
    {
      for (gsi = gsi_start_bb (bb); !gsi_end_p (gsi); gsi_next (&gsi))
	{
	  /* Pre-expand VEC_COND_EXPRs to .VCOND* internal function
	     calls mapping to supported optabs.  */
	  gimple *g = gimple_expand_vec_cond_expr (fun, &gsi,
						   &vec_cond_ssa_name_uses);
	  if (g != NULL)
	    {
	      tree lhs = gimple_assign_lhs (gsi_stmt (gsi));
	      gimple_set_lhs (g, lhs);
	      gsi_replace (&gsi, g, false);
	    }

	  /* Recognize .VEC_SET and .VEC_EXTRACT patterns.  */
	  cfg_changed |= gimple_expand_vec_set_extract_expr (fun, &gsi);
	  if (gsi_end_p (gsi))
	    break;

	  gassign *stmt = dyn_cast <gassign *> (*gsi);
	  if (!stmt)
	    continue;

	  tree_code code = gimple_assign_rhs_code (stmt);
	  tree lhs = gimple_assign_lhs (stmt);
	  if (TREE_CODE_CLASS (code) == tcc_comparison
	      && !has_single_use (lhs))
	    {
	      /* Duplicate COND_EXPR condition defs when they are
		 comparisons so RTL expansion with the help of TER
		 can perform better if conversion.  */
	      imm_use_iterator imm_iter;
	      use_operand_p use_p;
	      auto_vec<gassign *, 4> cond_exprs;
	      unsigned cnt = 0;
	      FOR_EACH_IMM_USE_FAST (use_p, imm_iter, lhs)
		{
		  if (is_gimple_debug (USE_STMT (use_p)))
		    continue;
		  cnt++;
		  if (gimple_bb (USE_STMT (use_p)) == bb
		      && is_gimple_assign (USE_STMT (use_p))
		      && gimple_assign_rhs1_ptr (USE_STMT (use_p)) == use_p->use
		      && gimple_assign_rhs_code (USE_STMT (use_p)) == COND_EXPR)
		    cond_exprs.safe_push (as_a <gassign *> (USE_STMT (use_p)));
		}
	      for (unsigned i = cond_exprs.length () == cnt ? 1 : 0;
		   i < cond_exprs.length (); ++i)
		{
		  gassign *copy = as_a <gassign *> (gimple_copy (stmt));
		  tree new_def = duplicate_ssa_name (lhs, copy);
		  gimple_assign_set_lhs (copy, new_def);
		  auto gsi2 = gsi_for_stmt (cond_exprs[i]);
		  gsi_insert_before (&gsi2, copy, GSI_SAME_STMT);
		  gimple_assign_set_rhs1 (cond_exprs[i], new_def);
		  update_stmt (cond_exprs[i]);
		}
	    }
	}
    }

  for (auto it = vec_cond_ssa_name_uses.begin ();
       it != vec_cond_ssa_name_uses.end (); ++it)
    bitmap_set_bit (dce_ssa_names, SSA_NAME_VERSION ((*it).first));

  simple_dce_from_worklist (dce_ssa_names);

  return cfg_changed ? TODO_cleanup_cfg : 0;
}

} // anon namespace

gimple_opt_pass *
make_pass_gimple_isel (gcc::context *ctxt)
{
  return new pass_gimple_isel (ctxt);
}
```

As we can the code is enclosed in an unnamed namespace, which will limit the visibility to the current transition unit. Then we define the `pass_data pass_data_gimple_isel` which is a structure that defines the metadata for the pass, it includes information such as the pass type(`GIMPLE_PASS`), name(`isel`), optimization group, and other properties.

Then the pass class `pass_gimple_isel` is defined which inherits from `gimple_opt_pass` which should be a bass class for passes operating on GIMPLE.

Then we have two main functions defined in the class which are:

* gate - It determines whether the pass should be executed on a given function. In this case, it always returns `true`, which means this pass should be executed on all functions.
    
* execute - This function contains the actual logic of the pass.
    

Lastly in the end, the `make_pass_gimple_isel` function returns an instance of `pass_gimple_isel`, which is used in the pass manager(`passes.cc`) to register the pass.

Now that we understand how passes are defined in the GCC compiler let's craft a pass that prints "Hello World".

```c
#include "config.h"
#include "system.h"
#include "coretypes.h"
#include "backend.h"
#include "tree-pass.h"
#include "gimple.h"

namespace {
  const pass_data pass_data_gimple_test = {
    GIMPLE_PASS,
    /* type */
    "test",
    /* name */
    OPTGROUP_NONE,
    /* optinfo_flags */
    TV_NONE,
    /* tv_id */
    0,
    /* properties_required */
    0,
    /* properties_provided */
    0,
    /* properties_destroyed */
    0,
    /* todo_flags_start */
    0 /* todo_flags_finish */
  };

  /* Test pass for dumping a test message. */
  class pass_test_dump: public gimple_opt_pass {
    public: pass_test_dump(gcc::context * ctxt): gimple_opt_pass(pass_data_gimple_test, ctxt) {}

    /* Pass gate function. */
    bool gate(function*) final override {
      return true;
    }

    /* Pass execution function. */
    unsigned int execute(function*) final override {
      fprintf(stderr, "Hello World\n");
      return 0;
    }
  };
}

/* Create the test dump pass. */
gimple_opt_pass * make_pass_test_dump(gcc::context * ctxt) {
  return new pass_test_dump(ctxt);
}
```

First, we define `pass_data_gimple_test` which is a constant that has metadata about our pass such as name(`test`), type(`GIMPLE_PASS`) etc. Then we define our `pass_test_dump` which inherits `gimple_opt_pass`, this class for now only outputs "Hello World" in the execute function, and the function always returns true as right now we are focusing on getting this test pass working at all times. Once it starts working we can customize it to only run when the `afmv` argument is passed.

In the end, we have our function `make_pass_test_dump` function which returns an instance of our class. Now we only have to do one thing we have to add our pass in the `passes.def` file and `tree-pass.h`.

Changes in the `passes.def` file:

We are only adding our test pass after our isel pass.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713769247456/37e70e8b-be4b-4d94-b66c-506ae721f2c7.png align="center")

Then we call our make function from the `tree-pass.h` file:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713769354020/e974fd95-7a28-4408-86b8-ab50153133a9.png align="center")

That's the changes that I have made for the diagnostic output, this pass that we made currently doesn't provide any diagnostic info as it depends on the other people's code mainly the cloning and the pruning passes. Once, we develop those passes we can inform the developer, how many clones were made, how many of those clones were pruned, and how much time it took for this whole process.

## **Sources**

Tyler, C. (n.d.). *Software portability and optimization*. [**matrix.senecapolytechnic.ca**](http://matrix.senecapolytechnic.ca/)**.**[**matrix.senecapolytechnic.ca/~chris.tyler/wi**](http://matrix.senecapolytechnic.ca/~chris.tyler/wi)**..**

*GCC, the GNU Compiler Collection - GNU Project*. (n.d.). [**https://gcc.gnu.org/**](https://gcc.gnu.org/)