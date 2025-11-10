# Designing an Online Music Streaming Service Like Spotify

## Question

Design an Online Music Streaming Service Like Spotify

## Requirements

1. The music streaming service should allow users to browse and search for songs, albums, and artists.
2. Users should be able to create and manage playlists.
3. The system should support user authentication and authorization.
4. Users should be able to play, pause, skip, and seek within songs.
5. The system should recommend songs and playlists based on user preferences and listening history.
6. The system should handle concurrent requests and ensure smooth streaming experience for multiple users.
7. The system should be scalable and handle a large volume of songs and users.
8. The system should be extensible to support additional features such as social sharing and offline playback.

## Solution

### Design overview

Order the system around the listener journey: discover → queue/playback → feedback (likes/skips) →
recommendations. Separate hot-path playback (low latency) from heavy read/search and background
recommendation jobs.

- Catalog: songs, albums, artists with indexing for search/browse.
- Playback: player/commands, queue management, and a streaming pipeline (buffering/seek).
- Personalization: recommendations driven by events (listens, likes, skips, follows).
- Social: follows and release notifications via Observer.
- Persistence behind repositories; singleton facades exist only where needed for demo simplicity.

### Core model

- Domain: `Song`, `Album`, `Artist`, `User`, `Playlist`
- Playback: `Player` with `PlayCommand`, `PauseCommand`, `NextTrackCommand`
- Library/Search: `MusicLibrary` or catalog service
- Personalization: `MusicRecommender` (strategy/pluggable models), events as input
- Orchestration: `MusicStreamingSystem` (facade): manages artists, users, playlists and player access

### Key flows

1. Discover/search: filter by title/artist; present albums/playlists
2. Playback: queue tracks; `Play/Pause/Next` commands control the `Player`
3. Feedback: likes/skips generate events for `MusicRecommender`
4. Social: artist releases notify followers (Observer); playlists can be shared

### Design Details

1. The **Song**, **Album**, and **Artist** classes represent the basic entities in the music streaming service, with properties such as ID, title, artist, album, duration, and relationships between them.
2. The **User** class represents a user of the music streaming service, with properties like ID, username, password, and a list of playlists.
3. The **Playlist** class represents a user-created playlist, containing a list of songs.
4. The **MusicLibrary** class serves as a central repository for storing and managing songs, albums, and artists. It follows the Singleton pattern to ensure a single instance of the music library.
5. The **UserManager** class handles user registration, login, and other user-related operations. It also follows the Singleton pattern.
6. The **MusicPlayer** class represents the music playback functionality, allowing users to play, pause, skip, and seek within songs.
7. The **MusicRecommender** class generates song recommendations based on user preferences and listening history. It follows the Singleton pattern.
8. The **MusicStreamingService** class is the main entry point of the music streaming service. It initializes the necessary components, handles user requests, and manages the overall functionality of the service.

### Implementation (Python)

#### artist.py

```python
from playable import Album
from typing import List
from observer import Subject

class Artist(Subject):
    def __init__(self, artist_id: str, name: str):
        super().__init__()
        self._id = artist_id
        self._name = name
        self._discography: List['Album'] = []

    def release_album(self, album: 'Album'):
        self._discography.append(album)
        print(f"[System] Artist {self._name} has released a new album: {album.get_title()}")
        self.notify_observers(self, album)

    @property
    def id(self) -> str:
        return self._id

    def get_name(self) -> str:
        return self._name
```

#### command.py

```python
from abc import ABC, abstractmethod
from player import Player

class Command(ABC):
    @abstractmethod
    def execute(self):
        pass

class PlayCommand(Command):
    def __init__(self, player: Player):
        self._player = player

    def execute(self):
        self._player.click_play()

class PauseCommand(Command):
    def __init__(self, player: Player):
        self._player = player

    def execute(self):
        self._player.click_pause()

class NextTrackCommand(Command):
    def __init__(self, player: Player):
        self._player = player

    def execute(self):
        self._player.click_next()
```

#### music_streaming_demo.py

```python
from music_streaming_system import MusicStreamingSystem
from artist import Artist
from playable import Album
from user import User
from subscription_tier import SubscriptionTier
from command import PlayCommand, PauseCommand, NextTrackCommand
from playable import Playlist

class MusicStreamingDemo:
    @staticmethod
    def main():
        system = MusicStreamingSystem.get_instance()

        # --- Setup Catalog ---
        daft_punk = Artist("art1", "Daft Punk")
        system.add_artist(daft_punk)

        discovery = Album("Discovery")
        s1 = system.add_song("s1", "One More Time", daft_punk.id, 320)
        s2 = system.add_song("s2", "Aerodynamic", daft_punk.id, 212)
        s3 = system.add_song("s3", "Digital Love", daft_punk.id, 301)
        s4 = system.add_song("s4", "Radioactive", daft_punk.id, 311)
        discovery.add_track(s1)
        discovery.add_track(s2)
        discovery.add_track(s3)
        discovery.add_track(s4)

        # --- Register Users (Builder Pattern) ---
        free_user = User.Builder("Alice").with_subscription(SubscriptionTier.FREE, 0).build()
        premium_user = User.Builder("Bob").with_subscription(SubscriptionTier.PREMIUM, 0).build()
        system.register_user(free_user)
        system.register_user(premium_user)

        # --- Observer Pattern: User follows artist ---
        print("--- Observer Pattern Demo ---")
        premium_user.follow_artist(daft_punk)
        daft_punk.release_album(discovery)  # This will notify Bob
        print()

        # --- Strategy Pattern: Playback behavior ---
        print("--- Strategy Pattern (Free vs Premium) & State Pattern (Player) Demo ---")
        player = system.get_player()
        player.load(discovery, free_user)

        # --- Command Pattern: Controlling the player ---
        play = PlayCommand(player)
        pause = PauseCommand(player)
        next_track = NextTrackCommand(player)

        play.execute()  # Plays song 1
        next_track.execute()  # Plays song 2
        pause.execute()  # Pauses song 2
        play.execute()  # Resumes song 2
        next_track.execute()  # Plays song 3
        next_track.execute()  # Plays song 4 (ad for free user)
        print()

        # --- Premium user experience (no ads) ---
        print("--- Premium User Experience ---")
        player.load(discovery, premium_user)
        play.execute()
        next_track.execute()
        print()

        # --- Composite Pattern: Play a playlist ---
        print("--- Composite Pattern Demo ---")
        my_playlist = Playlist("My Awesome Mix")
        my_playlist.add_track(s3)  # Digital Love
        my_playlist.add_track(s1)  # One More Time

        player.load(my_playlist, premium_user)
        play.execute()
        next_track.execute()
        print()

        # --- Search and Recommendation ---
        print("--- Search and Recommendation Service Demo ---")
        search_results = system.search_songs_by_title("love")
        print(f"Search results for 'love': {search_results}")

        recommendations = system.get_song_recommendations()
        print(f"Your daily recommendations: {recommendations}")

if __name__ == "__main__":
    MusicStreamingDemo.main()
```

#### music_streaming_system.py

```python
import threading
from typing import Dict, List
from user import User
from playable import Song
from artist import Artist
from player import Player
from search_service import SearchService
from recommendation_service import RecommendationService
from recommendation_strategy import GenreBasedRecommendationStrategy

class MusicStreamingSystem:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._initialized = False
        return cls._instance

    def __init__(self):
        if not self._initialized:
            self._users: Dict[str, User] = {}
            self._songs: Dict[str, Song] = {}
            self._artists: Dict[str, Artist] = {}
            self._player = Player()
            self._search_service = SearchService()
            self._recommendation_service = RecommendationService(GenreBasedRecommendationStrategy())
            self._initialized = True

    @classmethod
    def get_instance(cls):
        return cls()

    def register_user(self, user: User):
        self._users[user.id] = user

    def add_song(self, song_id: str, title: str, artist_id: str, duration: int) -> Song:
        song = Song(song_id, title, self._artists[artist_id], duration)
        self._songs[song.id] = song
        return song

    def add_artist(self, artist: Artist):
        self._artists[artist.id] = artist

    def search_songs_by_title(self, title: str) -> List[Song]:
        return self._search_service.search_songs_by_title(list(self._songs.values()), title)

    def get_song_recommendations(self) -> List[Song]:
        return self._recommendation_service.generate_recommendations(list(self._songs.values()))

    def get_player(self) -> Player:
        return self._player
```

#### observer.py

```python
from abc import ABC, abstractmethod
from typing import List
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from artist import Artist
    from playable import Album

class ArtistObserver(ABC):
    @abstractmethod
    def update(self, artist: 'Artist', new_album: 'Album'):
        pass

class Subject:
    def __init__(self):
        self._observers: List[ArtistObserver] = []

    def add_observer(self, observer: ArtistObserver):
        self._observers.append(observer)

    def remove_observer(self, observer: ArtistObserver):
        if observer in self._observers:
            self._observers.remove(observer)

    def notify_observers(self, artist: 'Artist', album: 'Album'):
        for observer in self._observers:
            observer.update(artist, album)
```

#### playable.py

```python
from abc import ABC, abstractmethod
from typing import List
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from artist import Artist

class Playable(ABC):
    @abstractmethod
    def get_tracks(self) -> List['Song']:
        pass

class Song(Playable):
    def __init__(self, song_id: str, title: str, artist: 'Artist', duration_in_seconds: int):
        self._id = song_id
        self._title = title
        self._artist = artist
        self._duration_in_seconds = duration_in_seconds

    def get_tracks(self) -> List['Song']:
        return [self]

    def __str__(self) -> str:
        return f"'{self._title}' by {self._artist.get_name()}"

    @property
    def id(self) -> str:
        return self._id

    @property
    def title(self) -> str:
        return self._title

    @property
    def artist(self) -> 'Artist':
        return self._artist

class Album(Playable):
    def __init__(self, title: str):
        self._title = title
        self._tracks: List[Song] = []

    def add_track(self, song: Song):
        self._tracks.append(song)

    def get_tracks(self) -> List[Song]:
        return self._tracks.copy()

    def get_title(self) -> str:
        return self._title

class Playlist(Playable):
    def __init__(self, name: str):
        self._name = name
        self._tracks: List[Song] = []

    def add_track(self, song: Song):
        self._tracks.append(song)

    def get_tracks(self) -> List[Song]:
        return self._tracks.copy()
```

#### playback_strategy.py

```python
from abc import ABC, abstractmethod
from subscription_tier import SubscriptionTier
from playable import Song
from player import Player
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from player import Player

class PlaybackStrategy(ABC):
    @abstractmethod
    def play(self, song: Song, player: 'Player'):
        pass

    @staticmethod
    def get_strategy(tier: SubscriptionTier, songs_played: int) -> 'PlaybackStrategy':
        if tier == SubscriptionTier.PREMIUM:
            return PremiumPlaybackStrategy()
        else:
            return FreePlaybackStrategy(songs_played)

class FreePlaybackStrategy(PlaybackStrategy):
    SONGS_BEFORE_AD = 3

    def __init__(self, initial_songs_played: int):
        self._songs_played = initial_songs_played

    def play(self, song: Song, player: 'Player'):
        if self._songs_played > 0 and self._songs_played % self.SONGS_BEFORE_AD == 0:
            print("\n>>> Playing Advertisement: 'Buy Spotify Premium for ad-free music!' <<<\n")
        player.set_current_song(song)
        print(f"Free User is now playing: {song}")
        self._songs_played += 1

class PremiumPlaybackStrategy(PlaybackStrategy):
    def play(self, song: Song, player: 'Player'):
        player.set_current_song(song)
        print(f"Premium User is now playing: {song}")
```

#### player.py

```python
from player_status import PlayerStatus
from playable import Playable, Song
from typing import List, Optional
from player_states import PlayerState, StoppedState
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from user import User

class Player:
    def __init__(self):
        self._state = StoppedState()
        self._status = PlayerStatus.STOPPED
        self._queue: List[Song] = []
        self._current_index = -1
        self._current_song: Optional[Song] = None
        self._current_user: Optional[User] = None

    def load(self, playable: Playable, user: 'User'):
        self._current_user = user
        self._queue = playable.get_tracks()
        self._current_index = 0
        print(f"Loaded {len(self._queue)} tracks for user {user.get_name()}.")
        self._state = StoppedState()

    def play_current_song_in_queue(self):
        if 0 <= self._current_index < len(self._queue):
            song_to_play = self._queue[self._current_index]
            self._current_user.playback_strategy.play(song_to_play, self)

    def click_play(self):
        self._state.play(self)

    def click_pause(self):
        self._state.pause(self)

    def click_next(self):
        if self._current_index < len(self._queue) - 1:
            self._current_index += 1
            self.play_current_song_in_queue()
        else:
            print("End of queue.")
            self._state.stop(self)

    def change_state(self, state: PlayerState):
        self._state = state

    def set_status(self, status: PlayerStatus):
        self._status = status

    def set_current_song(self, song: Song):
        self._current_song = song

    def has_queue(self) -> bool:
        return len(self._queue) > 0
```

#### player_states.py

```python
from abc import ABC, abstractmethod
from player_status import PlayerStatus
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from player import Player

class PlayerState(ABC):
    @abstractmethod
    def play(self, player: 'Player'):
        pass

    @abstractmethod
    def pause(self, player: 'Player'):
        pass

    @abstractmethod
    def stop(self, player: 'Player'):
        pass

class PausedState(PlayerState):
    def play(self, player: 'Player'):
        print("Resuming playback.")
        player.change_state(PlayingState())
        player.set_status(PlayerStatus.PLAYING)

    def pause(self, player: 'Player'):
        print("Already paused.")

    def stop(self, player: 'Player'):
        print("Stopping playback from paused state.")
        player.change_state(StoppedState())
        player.set_status(PlayerStatus.STOPPED)

class PlayingState(PlayerState):
    def play(self, player: 'Player'):
        print("Already playing.")

    def pause(self, player: 'Player'):
        print("Pausing playback.")
        player.change_state(PausedState())
        player.set_status(PlayerStatus.PAUSED)

    def stop(self, player: 'Player'):
        print("Stopping playback.")
        player.change_state(StoppedState())
        player.set_status(PlayerStatus.STOPPED)

class StoppedState(PlayerState):
    def play(self, player: 'Player'):
        if player.has_queue():
            print("Starting playback.")
            player.change_state(PlayingState())
            player.set_status(PlayerStatus.PLAYING)
            player.play_current_song_in_queue()
        else:
            print("Queue is empty. Load songs to play.")

    def pause(self, player: 'Player'):
        print("Cannot pause. Player is stopped.")

    def stop(self, player: 'Player'):
        print("Already stopped.")
```

#### player_status.py

```python
from enum import Enum

class PlayerStatus(Enum):
    PLAYING = "PLAYING"
    PAUSED = "PAUSED"
    STOPPED = "STOPPED"
```

#### recommendation_service.py

```python
from recommendation_strategy import RecommendationStrategy
from playable import Song
from typing import List

class RecommendationService:
    def __init__(self, strategy: RecommendationStrategy):
        self._strategy = strategy

    def set_strategy(self, strategy: RecommendationStrategy):
        self._strategy = strategy

    def generate_recommendations(self, all_songs: List[Song]) -> List[Song]:
        return self._strategy.recommend(all_songs)
```

#### recommendation_strategy.py

```python
from abc import ABC, abstractmethod
from typing import List
from playable import Song
import random

class RecommendationStrategy(ABC):
    @abstractmethod
    def recommend(self, all_songs: List[Song]) -> List[Song]:
        pass

class GenreBasedRecommendationStrategy(RecommendationStrategy):
    def recommend(self, all_songs: List[Song]) -> List[Song]:
        print("Generating genre-based recommendations (simulated)...")
        shuffled = all_songs.copy()
        random.shuffle(shuffled)
        return shuffled[:5]
```

#### search_service.py

```python
from typing import List
from playable import Song
from artist import Artist

class SearchService:
    def search_songs_by_title(self, songs: List[Song], query: str) -> List[Song]:
        return [s for s in songs if query.lower() in s.title.lower()]

    def search_artists_by_name(self, artists: List[Artist], query: str) -> List[Artist]:
        return [a for a in artists if query.lower() in a.get_name().lower()]
```

#### subscription_tier.py

```python
from enum import Enum

class SubscriptionTier(Enum):
    FREE = "FREE"
    PREMIUM = "PREMIUM"
```

#### user.py

```python
from observer import ArtistObserver
from typing import Set
from artist import Artist
from playable import Album
from subscription_tier import SubscriptionTier
from playback_strategy import PlaybackStrategy
import uuid

class User(ArtistObserver):
    def __init__(self, user_id: str, name: str, playback_strategy: PlaybackStrategy):
        self._id = user_id
        self._name = name
        self._playback_strategy = playback_strategy
        self._followed_artists: Set[Artist] = set()

    def follow_artist(self, artist: Artist):
        self._followed_artists.add(artist)
        artist.add_observer(self)

    def update(self, artist: Artist, new_album: Album):
        print(f"[Notification for {self._name}] Your followed artist {artist.get_name()} "
              f"just released a new album: {new_album.get_title()}!")

    @property
    def playback_strategy(self) -> PlaybackStrategy:
        return self._playback_strategy

    @property
    def id(self) -> str:
        return self._id

    def get_name(self) -> str:
        return self._name

    class Builder:
        def __init__(self, name: str):
            self._id = str(uuid.uuid4())
            self._name = name
            self._playback_strategy = None

        def with_subscription(self, tier: SubscriptionTier, songs_played: int) -> 'User.Builder':
            self._playback_strategy = PlaybackStrategy.get_strategy(tier, songs_played)
            return self

        def build(self) -> 'User':
            return User(self._id, self._name, self._playback_strategy)
```
