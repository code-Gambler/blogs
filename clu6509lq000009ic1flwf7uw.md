---
title: "Random Number Guessing Game  In Assembly"
seoTitle: "Number Guessing Game Written in 6502 Assembly"
seoDescription: "This is a 6502 Assembly number guessing game written by Steven David Pillay. The is my journey with the course Software Portability and Optimization"
datePublished: Sun Mar 24 2024 23:15:23 GMT+0000 (Coordinated Universal Time)
cuid: clu6509lq000009ic1flwf7uw
slug: random-number-guessing-game-in-assembly
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1711321853013/15a02c03-4447-41ea-a0f1-107217249134.webp
tags: spo600, lab2

---

Let's code a simple, random number guessing game in 6502 assembly language. The code that we are going to write is designed to work on the 6502 emulator, which can be found [here](https://matrix.senecapolytechnic.ca/~chris.tyler/6502/). If you want to know the specifics of the emulator you can refer to the notes found in the emulator itself or over [here](https://matrix.senecapolytechnic.ca/~chris.tyler/wiki/doku.php?id=spo600:6502_emulator).

Let's start, so basically our idea is to generate a random number between 1 and 10 and ask the user to guess the number, if the number is too high we are going to give the user the feedback, and vice versa. Also for this game, as there are 10 possible outcomes, we are going to give the user four chances, which is pretty generous. This is as if the user had four lives and he/she had to guess the right number to win. As the bitmap display is divided into four pages we are going to graphically display the number of lives remaining in the bitmap display.

## Code

```plaintext
; Constants
define NEWLINE $0d      ; ASCII code for newline character
define CONSTANT_NUMBER $0081; Memory location to store constant number
define remaining_guesses $0091
define guess $0095
define color $0097

start:
    LDA #$00    ; set a pointer in memory location $40 to point to $0200
    STA $40        ; ... low byte ($00) goes in address $40
    LDA #$02    
    STA $41        ; ... high byte ($02) goes into address $41
    LDA #$01    ; color number
    STA color
    STA $42
    LDY #$00    ; set index to 0
    lda $fe     ; Use the pseudo-random number generator
    cmp #$30                ; Check if it's a digit
    bcc start          ; If not, continue reading
    cmp #$40
    bcs start
    sta CONSTANT_NUMBER
    jsr initialize_game      ; Initialize the game
    jsr print_instructions   ; Print game instructions
    jsr input_guess          ; Input user's guess
    jsr input_loop
    jsr compare_guess        ; Compare the guess with the constant number
    jmp start                ; Repeat the game

initialize_game:
    lda #$04                  ; Initialize remaining guesses to 4
    sta remaining_guesses
    rts                      ; Return from subroutine

print_instructions:
    JSR PRINT
    dcb 10,"W","e","l","c","o","m","e",32,"t","o",32,"t","h","e",32,"n","u","m","b","e","r",32,"g","u","e","s","s","i","n","g",32,"g","a","m","e",10,00
    JSR PRINT
    dcb "Y","o","u",32,"h","a","v","e",32,"4",32,"g","u","e","s","s","e","s",32,"t","o",32 ,"g","u","e","s","s",32,"t","h","e",32,"n","u","m","b","e","r",".",10,00 
    JSR PRINT
    dcb "E","n","t","e","r",32,"y","o","u","r",32  ,"g","u","e","s","s",":",00
    rts                      ; Return from subroutine

input_guess:
    lda #$00                 ; Initialize guess variable
    sta guess
input_loop:
    lda #$00               ; Initialize accumulator to 0
read_char:
    jsr CHRIN           ; Get a character from input
    cmp #$29                ; Check if it's a digit
    bcc read_char          ; If not, continue reading
    cmp #$40
    bcs read_char
    ; Convert ASCII digit to binary and store in accumulator
    sta guess
    jsr CHROUT
end_input:
    jsr compare_guess                    ; Return from subroutine

compare_guess:
    lda CONSTANT_NUMBER  ; Load the address of the constant number
    ;lda (CONSTANT_NUMBER_ADDR)  ; Load the constant number from memory
    cmp guess               ; Compare with user's guess
    beq correct_guess       ; If equal, go to correct_guess
    bcs too_low            ; If constant number > guess, go to too_high
    jmp too_high             ; Otherwise, go to too_low

too_low:
    JSR PRINT
    dcb 10,"T","o","o",32,"l","o","w","!",00
    dec remaining_guesses   ; Decrement remaining guesses
    bne reask         ; If remaining guesses, get another guess
    jmp end_game            ; Otherwise, end the game

too_high:
    JSR PRINT
    dcb 10,"T","o","o",32,"H","i","g","h"!",00
    dec remaining_guesses   ; Decrement remaining guesses
    bne reask         ; If remaining guesses, get another guess
    jmp end_game            ; Otherwise, end the game

correct_guess:
    JSR PRINT
    dcb 10,"C","o","n","g","r","a","t","u","l","a","t","i","o","n","s","!",32,"Y","o","u",32,"g","u","e","s","s","e","d",32,"t","h","e",32,"n","u","m","b","e","r","!",00
    jmp end_game            ; End the game

reask:
    JSR LOOP
    JSR PRINT
    dcb 10,"E","n","t","e","r",32 ,"a","n","o","t","h","e","r",32,"g","u","e","s","s",":",00
    JSR input_loop
end_game:
    JSR LOOP
    JSR PRINT
    dcb 10,"T","h","e",32,"n","u","m","b","e","r",32 ,"w","a","s",":",00
    lda CONSTANT_NUMBER  ; Load the address of the constant number
    jsr CHROUT
    lda #NEWLINE
    jsr CHROUT
    brk
    rts
; ROM routines
define SCINIT $ff81       ; Initialize/clear screen
define CHRIN $ffcf        ; Input character from keyboard
define CHROUT $ffd2       ; Output character to screen
define SCREEN $ffed       ; Get screen size
define PLOT $fff0         ; Get/set cursor coordinates

LOOP:
     LDA color    
     STA ($40),y    ; set pixel color at the address (pointer)+Y
     INY            ; increment index
     BNE LOOP    ; continue until done the page (256 pixels)
     INC $41        ; increment the page
     LDX $41        ; get the current page number
     rts

; zeropage variables
define		PRINT_PTR	$00
define		PRINT_PTR_H	$01
define		CURRENT		$02
define		SCRN_PTR	$03
define		SCRN_PTR_H	$04
;Author: Chris Tyler, Src: https://github.com/ctyler/6502js-code/blob/master/colour-selector-live.6502
; --------------------------------------------------------
; Print a message
; 
; Prints the message in memory immediately after the 
; JSR PRINT. The message must be null-terminated and
; 255 characters maximum in length.

PRINT:		pla
		clc
		adc #$01
		sta PRINT_PTR
		pla
		sta PRINT_PTR_H

		tya
		pha

		ldy #$00
print_next:	lda (PRINT_PTR),y
		beq print_done
		
		jsr CHROUT
		iny
		jmp print_next

print_done:	tya
		clc
		adc PRINT_PTR
		sta PRINT_PTR

		lda PRINT_PTR_H
		adc #$00
		sta PRINT_PTR_H

		pla
		tay

		lda PRINT_PTR_H
		pha
		lda PRINT_PTR
		pha

		rts
```

## Breakdown

The breakdown of the code is as follows:

* Define the constants first.
    
* Initialize the bitmap display and generate the random number for the game.
    
* Print the rules of the game for the user.
    
* Take only numerical input from the user.
    
* Compare it with the generated number and react accordingly.
    
* In the end, we have the borrowed code from my professor's [repository](https://github.com/ctyler/6502js-code/blob/master/colour-selector-live.6502).
    

The program achieves the following:

1. Works perfectly on the 6502 emulator.
    
2. Outputs character to the screen and the graphical representation of the Lives on the bitmap display.
    
3. Accepts numerical input from the user.
    
4. Use mathematical instructions to calculate the number of remaining guesses.
    

## Limitations

This program, sometimes generates a character like "?" or ";" as its random number, I already have check right after the random number is guessed but somehow it passes the check. If you figured out the problem, feel free to create a PR or open an Issue [here](https://github.com/code-Gambler/6502-Assembly).

## Demo

%[https://youtu.be/ZF1MXNJTzRc] 

The most updated version of this code can be found in my [GitHub](https://github.com/code-Gambler/6502-Assembly), feel free to contribute new features or improvements by opening PRs.

Sources:

Tyler, C. (n.d.). *Software portability and optimization*. [**matrix.senecapolytechnic.ca**](http://matrix.senecapolytechnic.ca/). [**https://matrix.senecapolytechnic.ca/~chris.tyler/wiki/doku.php?id=spo600:start**](https://matrix.senecapolytechnic.ca/~chris.tyler/wiki/doku.php?id=spo600:start)