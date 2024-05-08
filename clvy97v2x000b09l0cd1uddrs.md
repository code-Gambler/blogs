---
title: "Movie Recommendation System"
datePublished: Wed May 08 2024 20:10:31 GMT+0000 (Coordinated Universal Time)
cuid: clvy97v2x000b09l0cd1uddrs
slug: movie-recommendation-system
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1715199002382/f7e70d33-4c53-4aa8-94b6-fd4a6153c44a.jpeg
tags: machine-learning, ml, movies

---

I am going to be developing a Movie Recommendation System using Python(Numpy, Panadas and Flask). The first step to developing this system is understanding the dataset.

## DataSet Analysis

For this Project, I will be using the dataset from [GroupLens](https://grouplens.org/datasets/movielens/latest/). At first, we will be using the small dataset and once we are confident with the Recommendation model we will upgrade it to the Full DataSet. Now let us first understand the data. After briefly observing the Dataset I realized that all these files are interconnected via the primary Key `movieId`. The zip file has Five files namely:

* links - A CSV file that links the movie ID to the IMDB and TMDB IDs. This can be useful to provide the users with a link to the IMDB and TMDB webpages when recommending the movie.
    
* movies - A CSV file that contains the title and genre of the movie.
    
* ratings - A CSV file that contains all the ratings of the user.
    
* tags - A CSV file that contains all the tags applied to a movie by a user.
    
* README - A readme file that provides information about the Content Arrangement, Licenses, etc.
    

### Exploratory data analysis

I will be using pandas profiling to generate EDA reports for our Data. The report will provide us with some essential information like missing values, duplicate rows, mean, maximum, minimum etc.

### `movies.csv`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1715030723676/1ebb99c7-a3b4-4b8c-afea-2718fd1a4eba.png align="center")

### `ratings.csv`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1715031244581/1d2e7cde-67ee-4089-81f8-c7d8d31b41e6.png align="center")

### `links.csv`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1715031332134/2b40df07-e8bf-4a19-add1-112d77a77d67.png align="center")

### `tags.csv`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1715031426659/ca74737d-72ea-4060-a92d-86422f9114ba.png align="center")

This analysis clearly shows that we do have missing or duplicate data in all the CSV files except `links.csv.` After having a more thorough look I figured out that some of the movies didn't have `tmdbId`.

| movieId | imdbId | tmdbId |
| --- | --- | --- |
| 791 | 0113610 |  |
| 1107 | 0102336 |  |
| 2851 | 0081454 |  |
| 4051 | 0056600 |  |
| 26587 | 0092337 |  |
| 32600 | 0377059 |  |
| 40697 | 0105946 |  |
| 79299 | 0874957 |  |

If I were to return links to the web pages of the movies via the API, I would probably use IMDB instead of TMDB, so it is not that big of a problem.

All these reports can be generated using `generate_report()` function present in the `recommend-logic/generate_eda_report.py` file.

## Data Preprocessing

Now that I analyzed the data, I cleaned the title by removing everything from the title except alphanumeric characters and spaces. Then I stored this title under a separate column called "clean title".

```python
def clean_title(title):
    """Cleans title of the movies file by removing all the character except
       space and alphanumeric characters.

    Args:
        title: The title of the movie to clean.

    Returns:
        The cleaned title string.
    """
    title = re.sub("[^a-zA-Z0-9 ]", "", title)
    return title

# Preprocessing
movies["clean_title"] = movies["title"].apply(clean_title)
```

After that, I needed to vectorize the movie title so that I could create a search engine to look up the title of the movie. I vectorized the data using **Tfidf-Vectorizer** from the sci-kit-learn library.

## Search Function

Now that I have vectorized my movie title, I need to create a search function that would look up movie titles and return the appropriate movie titles. I used the cosine similarity function from the sci-kit-learn library to compare and return the movies whose title closely matches the given title.

```python
def search(title):
    """Searches for similar movies based on the provided title using TF-IDF vectorizer.

    Args:
        title: The title of the movie to search.

    Returns:
        A pandas DataFrame containing the top 5 most similar movies based on title.
    """
    title = clean_title(title)
    query_vec = vectorizer.transform([title])
    similarity = cosine_similarity(query_vec, vectorized_movies).flatten()
    indices = np.argpartition(similarity, -5)[-5:]
    results = movies.iloc[indices].iloc[::-1]
    return results
```

This search function looks up the movies and returns the 5 most matched movie titles.

## User Preferences

There were two main solutions that I could implement collaborative filtering and content-based filtering. Here is an analogy to understand the difference between both.

Collaborative filtering is like asking a friend or yours who has a similar taste in movies for recommendations and Content-based filtering is like researching what is the genre, plot of the movie, cast and crew etc of the movies and then deciding what to watch

I will be implementing both solutions. The user can specify a movie that they like and they will get a JSON response containing similar movies or specify genres that they and the API will in turn send a JSON response of the highest-rated movies in those genres.

## Recommendation Logic

### Collaborative Filtering

Now when a client provides a movie that they like, first we search the list of movies using our search function which will return the closest match, then will find out all the users that rated that movie a rating of 4 or above. Now after we have done that we will make a list of movies that those users have rated more than 4. Now we have a list of movies that users who are similar to the client like. After this, we find the ratings for these movies by all the users. Then we have two metrics ratings by users similar to the client and ratings by all the users, now we calculate a score by subtracting both these values and then arranging them in descending order. Now we have a list of movies arranged in descending values of their score that the client may like. After this, we return the top 10 movies along with the IMDB link for the movies in JSON format to the client.

```python
def find_similar_movies(movie_id):
    """Finds similar movies based on a combination of user ratings and movie content.

    This function first finds movies recommended based on similar user ratings for the provided movie_id.
    Then, it calculates a score indicating the recommendation strength based on the difference between
    recommendation scores from similar users and overall user ratings. Finally, it retrieves details
    like title, genres, and imdb link for the top 10 recommended movies.

    Args:
        movie_id: The ID of the movie to use as a base for recommendations.

    Returns:
        A pandas DataFrame containing details (title, genres, imdb link, score) for the top 10 recommended movies.
    """
    similar_user_recs = find_sim_user_recs(movie_id)
    all_user_recs = find_all_user_recs(similar_user_recs)
    percentage_difference = find_percentage_difference(similar_user_recs, all_user_recs)
    return percentage_difference.head(10).merge(movies, left_index=True, right_on="movieId").merge(links)[["score", "title", "genres", "imdbId"]]

def get_recommendations(title):
    """Recommends similar movies based on user ratings and content.

    This function searches for movies similar to the provided title, finds recommendations based on
    a combination of user ratings and movie content for the most similar movie, and converts
    the results to a JSON dictionary.

    Args:
        title: The title of the movie to use for recommendations.

    Returns:
        A dictionary containing a list of recommended movies with details
        including title, genres, imdb link, and a score indicating recommendation
        strength.
    """
    results = search(title)
    movie_id = results.iloc[0]["movieId"]
    similar_movies = find_similar_movies(movie_id)
    similar_movies_with_link = covert_imdb_id_to_link(similar_movies)
    similar_movies_with_link = remove_element(similar_movies_with_link, 0)
    return covert_to_json(similar_movies_with_link)
```

### Content-Based Filtering

For this, when the client first provides the genres they like, we first calculate the mean rating for all the movies and filter out all the movies based on the genres provided, then we arrange the movies in descending order with respect to the mean rating and return the top 10 movies along with the IMDB link for movies in JSON format.

```python
def get_movies_by_genre(movies, genre):
    """Filters movies based on a specific genre.

    This function creates a new boolean column indicating whether a movie belongs to the specified genre
    based on case-insensitive string matching. It then filters the DataFrame to include only movies
    where the boolean column is True. Finally, it merges the results with the links DataFrame
    to include the 'imdbId' for each movie.

    Args:
        movies: A pandas DataFrame containing movie data.
        genre: The genre to filter movies by (case-insensitive).

    Returns:
        A pandas DataFrame containing details (title, genres, imdb link) for movies belonging to the specified genre.
    """
    movies["boolean"] = movies['genres'].apply(lambda x: 1 if all(i.casefold() in x.casefold() for i in genre) else 0)
    recommendation_by_genre = movies[(movies.boolean == 1)]
    return recommendation_by_genre.head(10).merge(links)[["title", "genres", "imdbId"]]

def get_recommendations_by_genre(genre):
    """Recommends movies based on a specified genre.

    This function first retrieves the top movies with high average ratings using `get_average_rating_desc`.
    Then, it filters those movies based on the provided genre using `get_movies_by_genre`. Finally,
    it converts the recommendations to JSON format using `covert_to_json`.

    Args:
        genre: The genre to recommend movies from (case-insensitive).

    Returns:
        A dictionary containing a list of recommended movies with details
        including title, genres, imdb link, in JSON format.
    """
    average_rating_desc = get_average_rating_desc()
    recommendation_by_genre = get_movies_by_genre(average_rating_desc, genre)
    recommendation_by_genre_with_links = covert_imdb_id_to_link(recommendation_by_genre)
    return covert_to_json(recommendation_by_genre_with_links)
```

Now as the movie recommendation logic is ready let's move on to developing the Flask API.

## Flask API

This API is going to the following endpoints:

### `/`

Health route check, which will return a JSON object containing the author name, GitHub URL and name of the host running the application.

### `/api/v1/recommend`

If this route receives the title of the movie that the client likes as the query parameter, it will use the Collaborative Filtering logic

But if this route receives the genres of the movies that the user would like to watch, it will use the Content-Based Filtering logic.

I am going to keep this API route versioned as well, so if we decide to add some features that are going to break the current version, we can still support the clients using the previous versions

### Tests

After developing this API, I wrote 3 simple tests that check the following:

* Heath route:
    
    ```python
    import json
    import sys
    from pathlib import Path
    sys.path.append(str(Path(__file__).parent.parent))
    import app
    
    def test_health_check():
        response = app.create_app().test_client().get('/')
        json_dict = json.loads(response.data)
        assert response.status_code == 200
        assert json_dict["author"] == "Steven David Pillay"
        assert json_dict["githubUrl"] == "https://github.com/code-Gambler/movie-recommender-api"
    ```
    
* Title Recommendation (Collaborative Filtering)
    
    ```python
    import sys
    from pathlib import Path
    sys.path.append(str(Path(__file__).parent.parent))
    import app
    
    def test_health_check():
        response = app.create_app().test_client().get('/api/v1/recommend?title=interstellar%202014')
        assert response.status_code == 200
        assert response.data == b'[{"genres":"Adventure|Drama","imdbLink":"http://www.imdb.com/title/tt1663202/","score":7.4521859204,"title":"The Revenant (2015)"},{"genres":"Action|Sci-Fi|IMAX","imdbLink":"http://www.imdb.com/title/tt1454468/","score":7.3843153457,"title":"Gravity (2013)"},{"genres":"Adventure|Drama|Sci-Fi","imdbLink":"http://www.imdb.com/title/tt3659388/","score":7.1828211405,"title":"The Martian (2015)"},{"genres":"Action|Sci-Fi|IMAX","imdbLink":"http://www.imdb.com/title/tt1631867/","score":7.1515312111,"title":"Edge of Tomorrow (2014)"},{"genres":"Sci-Fi","imdbLink":"http://www.imdb.com/title/tt2543164/","score":7.0998370674,"title":"Arrival (2016)"},{"genres":"Drama|Sci-Fi|Thriller","imdbLink":"http://www.imdb.com/title/tt0470752/","score":6.9756378285,"title":"Ex Machina (2015)"},{"genres":"Drama|Thriller|War","imdbLink":"http://www.imdb.com/title/tt2084970/","score":6.4625847617,"title":"The Imitation Game (2014)"},{"genres":"Drama|Thriller","imdbLink":"http://www.imdb.com/title/tt2267998/","score":6.3880750782,"title":"Gone Girl (2014)"},{"genres":"Sci-Fi|Thriller","imdbLink":"http://www.imdb.com/title/tt1219289/","score":6.3203225396,"title":"Limitless (2011)"}]\n'
    ```
    
* Genre Recommendation (Content-Based Filtering)
    
    ```python
    import sys
    from pathlib import Path
    sys.path.append(str(Path(__file__).parent.parent))
    import app
    
    def test_health_check():
        response = app.create_app().test_client().get('/api/v1/recommend?genre=sci-fi,drama')
        assert response.status_code == 200
        assert response.data == b'[{"genres":"Comedy|Drama|Sci-Fi","imdbLink":"http://www.imdb.com/title/tt2040470/","title":"Plato\'s Reality Machine (2013)"},{"genres":"Action|Drama|Mystery|Romance|Sci-Fi|Thriller","imdbLink":"http://www.imdb.com/title/tt0288808/","title":"Say Nothing (2001)"},{"genres":"Drama|Horror|Sci-Fi","imdbLink":"http://www.imdb.com/title/tt0071855/","title":"Moonchild (1974)"},{"genres":"Drama|Horror|Sci-Fi|Thriller","imdbLink":"http://www.imdb.com/title/tt0781392/","title":"Black Night (2006)"},{"genres":"Drama|Romance|Sci-Fi","imdbLink":"http://www.imdb.com/title/tt1894616/","title":"Awaken (2013)"},{"genres":"Drama|Sci-Fi","imdbLink":"http://www.imdb.com/title/tt3444016/","title":"The Awareness (2014)"},{"genres":"Documentary|Drama|Sci-Fi","imdbLink":"http://www.imdb.com/title/tt1590010/","title":"Voyage to Metropolis (2010)"},{"genres":"Drama|Mystery|Sci-Fi","imdbLink":"http://www.imdb.com/title/tt7243006/","title":"Meteors (2017)"},{"genres":"Adventure|Drama|Sci-Fi|Thriller","imdbLink":"http://www.imdb.com/title/tt0069601/","title":"Failure of Engineer Garin (1973)"},{"genres":"Action|Crime|Drama|Mystery|Sci-Fi|Thriller|IMAX","imdbLink":"http://www.imdb.com/title/tt1375666/","title":"Inception (2010)"}]\n'
    ```
    

### Docker

After this, I created a `Dockerfile` and Dockerised my API.

```dockerfile
FROM python:3.12-slim-bookworm

WORKDIR /python-docker

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .
EXPOSE 5000

CMD [ "python", "api/app.py"]
# We run our service on port 5000 by default
```

Then I built the image and pushed it to a [DockerHub](https://hub.docker.com/repository/docker/stevenpillay/movie-recommender-api/general).

## GitHub

The GitHub repository can be found at [https://github.com/code-Gambler/movie-recommender-api](https://github.com/code-Gambler/movie-recommender-api). It also has all the documentation on how to use the API and how to start a development server locally as well as using Docker.

## Sources

[https://repository-images.githubusercontent.com/275336521/20d38e00-6634-11eb-9d1f-6a5232d0f84f](https://repository-images.githubusercontent.com/275336521/20d38e00-6634-11eb-9d1f-6a5232d0f84f)