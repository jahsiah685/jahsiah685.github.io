import lyricsgenius as genius
from Phyme import Phyme
import re

# =================================================================
# 1. SETUP: Replace with your actual Genius Client Access Token
# =================================================================
GENIUS_TOKEN = "YOUR_GENIUS_TOKEN" 
ph = Phyme()
api = genius.Genius(GENIUS_TOKEN, remove_section_headers=True, verbose=False)

def find_rhyming_songs(artist_name, target_word, max_songs=10):
    """
    Finds songs by an artist that contain a word that rhymes with the target_word.
    """
    
    # -----------------------------------------------
    # 2. RHYME GENERATION: Find words that perfectly rhyme
    # -----------------------------------------------
    try:
        # Get a list of words that perfectly rhyme with the target word
        rhyme_dict = ph.get_perfect_rhymes(target_word.lower())
        
        # Phyme returns a dictionary like {'exact': ['car', 'far', 'jar']}
        # We flatten this into a single set for quick lookup
        rhyming_words = set(rhyme for rhymes in rhyme_dict.values() for rhyme in rhymes)
        
    except KeyError:
        print(f"⚠️ Could not find perfect rhymes for '{target_word}'. Check spelling.")
        return
    
    # Also include the original word in the set
    rhyming_words.add(target_word.lower())
    
    print(f"--- Searching for: '{target_word}' (and its rhymes) in {artist_name} lyrics ---")
    print(f"Rhyming words generated: {list(rhyming_words)[:5]}... ({len(rhyming_words)} total)")
    
    # -----------------------------------------------
    # 3. LYRICS ACQUISITION: Search and collect lyrics
    # -----------------------------------------------
    try:
        # Search the artist and grab their songs
        artist = api.search_artist(artist_name, max_songs=max_songs)
    except Exception as e:
        print(f"An error occurred fetching the artist data: {e}")
        return

    # -----------------------------------------------
    # 4. SEARCH AND REPORT
    # -----------------------------------------------
    found_songs = []
    
    for song in artist.songs:
        # Pre-process lyrics: lower case, remove punctuation, and split into words
        lyrics_text = song.lyrics.lower()
        
        # Use a regex to find all matches (whole word only)
        # re.escape handles any special characters in the rhyming words
        pattern = r'\b(' + '|'.join(re.escape(word) for word in rhyming_words) + r')\b'
        matches = re.findall(pattern, lyrics_text)
        
        if matches:
            # Get the unique rhyming words found in the song
            unique_matches = set(matches)
            found_songs.append({
                'title': song.title,
                'artist': artist_name,
                'rhymes_found': unique_matches
            })

    # -----------------------------------------------
    # 5. OUTPUT RESULTS
    # -----------------------------------------------
    if found_songs:
        print("\n✅ **Songs Found:**")
        for song in found_songs:
            print(f"   - {song['title']} by {song['artist']}")
            print(f"     -> Words found: {', '.join(song['rhymes_found'])}")
    else:
        print("\n❌ No songs found containing the target word or its perfect rhymes.")

# =================================================================
# 6. RUN THE SEARCH
# =================================================================

# Example 1: Search for 'fire' by The Beatles
# find_rhyming_songs("The Beatles", "fire", max_songs=5)

# Example 2: Search for 'dream' by a popular rapper
# find_rhyming_songs("Eminem", "dream", max_songs=10)

# Example 3: Search for 'day' by Taylor Swift
find_rhyming_songs("Taylor Swift", "day", max_songs=15)
