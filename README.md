# movie-search-engine
import requests
import json
import sqlite3

def get_movie_data(movie_name):
    api_key = "485a9f15"
    url = f"http://www.omdbapi.com/?t={movie_name}&apikey={api_key}"
    response = requests.get(url)
    data = response.json()
    if data.get("Response") == "True":
        return {
            "Title": data.get("Title", "N/A"),
            "Released": data.get("Released", "N/A"),
            "Genre": data.get("Genre", "N/A"),
            "Director": data.get("Director", "N/A"),
            "Actors": data.get("Actors", "N/A"),
            "Plot": data.get("Plot", "N/A"),
            "IMDb Rating": data.get("imdbRating", "N/A")
        }
    else:
        return {"Error": data.get("Error", "Something went wrong!")}

def add_to_favourites(movie_name):
    conn = sqlite3.connect("movies.db")
    cursor = conn.cursor()
    cursor.execute("INSERT INTO favourites (movie_name) VALUES (?)", (movie_name,))
    conn.commit()
    conn.close()

def view_search_history():
    conn = sqlite3.connect("movies.db")
    cursor = conn.cursor()
    cursor.execute("SELECT movie_name FROM search_history")
    rows = cursor.fetchall()
    conn.close()
    return [row[0] for row in rows]

def remove_from_favourites(movie_id):
    conn = sqlite3.connect("movies.db")
    cursor = conn.cursor()
    cursor.execute("DELETE FROM favourites WHERE id = ?", (movie_id,))
    conn.commit()
    conn.close()

def view_favourites():
    conn = sqlite3.connect("movies.db")
    cursor = conn.cursor()
    cursor.execute("SELECT id, movie_name FROM favourites")
    rows = cursor.fetchall()
    conn.close()
    return rows

def display_movie_details(details):
    if "Error" in details:
        print(f"\nError: {details['Error']}\n")
    else:
        print("\nMovie Details:")
        for key, value in details.items():
            print(f"{key}: {value}")
        print()

def initialize_database():
    conn = sqlite3.connect("movies.db")
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS search_history (
            id INTEGER PRIMARY KEY,
            movie_name TEXT
        )
    """)
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS favourites (
            id INTEGER PRIMARY KEY,
            movie_name TEXT
        )
    """)
    conn.commit()
    conn.close()

def add_to_search_history(movie_name):
    conn = sqlite3.connect("movies.db")
    cursor = conn.cursor()
    cursor.execute("INSERT INTO search_history (movie_name) VALUES (?)", (movie_name,))
    conn.commit()
    conn.close()

def main():
    initialize_database()
    while True:
        print("\nMovie Search App")
        print("1. Search for a movie")
        print("2. View search history")
        print("3. View favorite movies")
        print("4. Exit")
        choice = input("Enter your choice (1/2/3/4): ").strip()

        if choice == "1":
            movie_name = input("\nEnter the name of the movie (or 'movie_name fav' to mark favorite): ").strip()
            if not movie_name:
                print("Please enter a valid movie name.")
                continue

            if movie_name.endswith(" fav"):
                movie_name = movie_name.removesuffix(" fav").strip()
                add_to_favourites(movie_name)
                print(f"'{movie_name}' has been added to favourites.")
                continue

            movie_details = get_movie_data(movie_name)
            add_to_search_history(movie_name)
            display_movie_details(movie_details)

        elif choice == "2":
            print("\nSearch History:")
            history = view_search_history()
            if history:
                i = 1
                for movie in history:
                    print(f"{i}. {movie}")
                    i += 1
            else:
                print("No search history found.")

        elif choice == "3":
            print("\nFavorite Movies:")
            favourites = view_favourites()
            if favourites:
                for fav in favourites:
                    print(f"{fav[0]}. {fav[1]}")
                action = input("\nEnter the ID of a movie to remove it from favourites, or press Enter to go back: ").strip()
                if action.isdigit():
                    remove_from_favourites(int(action))
                    print("Movie removed from favourites.")
            else:
                print("No favorite movies found.")

        elif choice == "4":
            print("\nThank you for using the Movie Search App. Goodbye!")
            break
        else:
            print("Invalid choice. Please try again.")

if __name__ == "__main__":
    main()

