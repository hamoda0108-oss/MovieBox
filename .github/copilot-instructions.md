# MovieBox Copilot Instructions

## Project Overview

MovieBox is an iOS educational project demonstrating three architectural patterns (MVC, MVVM, VIPER) for building a movie list application. Each pattern is implemented as a separate Xcode target sharing common APIs and utilities.

**Key Repo Structure:**
- `MovieBox/` - Main MVC implementation with storyboards
- `MovieBoxAPI/` - Shared API layer (TopMoviesService, DTOs, Alamofire networking)
- `MovieBox.xcworkspace/` - Workspace managing all targets
- `Commons/` - Shared presentations and test utilities across architectures
- `Utility/` - Shared Swift extensions (Array, Optional helpers)
- `MovieBoxMVVM/` and `MovieBoxVIPER/` - Alternative architecture implementations (same scenes, different patterns)

## Architecture & Data Flow

### Layer Separation
1. **MovieBoxAPI Framework** (shared dependency)
   - Handles all network calls via `TopMoviesService`
   - Defines DTOs (`Movie`, `Genre`) with custom Codable mappings
   - Error handling through `Result` enum wrapper
   - Uses Alamofire for HTTP requests (fetches iTunes top 25 movies JSON)

2. **Presentation Layer** (`Commons/Presentations/`)
   - `MoviePresentation` - Simple DTO for UI display (title, detail)
   - Extensions (`MoviePresentation+API.swift`) convert API `Movie` → `MoviePresentation`
   - Instances are NSObject with equality overriding for testing

3. **MVC Targets** (MovieBox app)
   - ViewControllers manage state and service calls directly
   - Builders pattern for scene construction (`MovieDetailBuilder.make()`)
   - Storyboards define UI structure (see `MovieDetail.storyboard`)

### Key Data Transformation
```
TopMoviesService.fetchTopMovies() 
  → TopMoviesResponse.results: [Movie]
  → MoviePresentation.init(movie:) converts to UI model
  → ViewControllers update UI via custom views
```

## Build & Setup

**Dependency Management:**
- Uses Carthage for dependency resolution (see `Vendor/Cartfile`)
- Run `make carthage_bootstrap` after cloning to fetch frameworks
- Workspace auto-links frameworks to all targets

**Xcode Build:**
- Open `MovieBox.xcworkspace` (NOT .xcodeproj)
- Three targets with same code structure: MovieBoxMVC, MovieBoxMVVM, MovieBoxVIPER
- Each has Info.plist, AppDelegate, AppContainer, AppRouter for routing

## Testing Patterns

**Test Structure:**
- `Commons/Test/` provides shared test utilities
- `MockTopMoviesService` in Commons used across all architecture tests
- `ResourceLoader` loads JSON fixtures from `Commons/Test/Resources/` (movie1.json, movie2.json, etc.)

**Testing Approach (from MovieBoxMVCTests):**
- Use mock views and services injected into controllers
- Assert loading state changes: `XCTAssertEqual(view.isLoadingValues, [true, false])`
- Compare presentations: `view.movieList?.element(at: 0).title`
- Use `@testable` import to access internal types

```swift
// Pattern from MovieBoxMVCTests.swift
override func setUp() {
    service = MockTopMoviesService()
    view = MockMovieListView()
    controller = MovieListViewController()
    controller.customView = view
    controller.service = service
}
```

## Project-Specific Conventions

1. **Service Injection** - Services passed to ViewControllers before use:
   ```swift
   controller.service = service  // TopMoviesServiceProtocol required
   ```

2. **Error Handling** - TopMoviesService wraps errors:
   ```swift
   public enum Error: Swift.Error {
       case serializationError(internal: Swift.Error)
       case networkError(internal: Swift.Error)
   }
   ```

3. **Scene Building** - Use Builder pattern with storyboards:
   ```swift
   // MovieDetailBuilder.make(with: movie) returns configured ViewController
   // Loads from storyboard, injects Movie dependency
   ```

4. **Presentation Mapping** - Extensions handle API→UI conversion:
   - Never reference MovieBoxAPI types in ViewControllers
   - Use `MoviePresentation+API.swift` extension pattern to maintain separation

5. **Commons as Shared** - Only place truly cross-target code there:
   - Presentations used by all three architectures
   - Test mocks and fixtures
   - Extension helpers from Utility framework

## Cross-Component Communication

- **ViewControllers → Services**: Dependency injection via properties
- **Services → ViewControllers**: Callbacks using `@escaping` closures with Result enum
- **Routing**: AppRouter and AppContainer manage navigation (see App/ directories)
- **Multiple Architectures**: Each pattern (MVC/MVVM/VIPER) uses same AppContainer structure but different scene implementation

## External Dependencies

- **Alamofire** - HTTP networking (via Carthage)
- **Foundation** - JSON decoding, date handling via `Decoders.plainDateDecoder`
- **UIKit** - All UI implementations

## When Making Changes

1. **Bug fixes in API logic**: Edit `MovieBoxAPI/TopMoviesService.swift`
2. **Adding new screens**: Create in target-specific `Scenes/` folder
3. **Cross-architecture logic**: Put in `Commons/` or `Utility/`
4. **Presentation changes**: Update `MoviePresentation.swift` or extensions
5. **Tests**: Use mocking pattern from `MovieBoxMVCTests` and inject dependencies
