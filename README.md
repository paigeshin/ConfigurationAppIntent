# ConfigurationAppIntent

```swift
import WidgetKit
import AppIntents

struct ConfigurationAppIntent: WidgetConfigurationIntent {
    static var title: LocalizedStringResource = "Select Character"
    static var description = IntentDescription("Selects the character to display information for.")
    
    @Parameter(title: "Title", default: "A Default Title")
    var title: String
    
    @Parameter(title: "Character")
    var character: CharacterDetail
    
    
    init(character: CharacterDetail) {
        self.character = character
    }
    
    
    init() {
    }
}

struct CharacterDetail: AppEntity {
    let id: String
    var avatar: String
    let healthLevel: Double
    let heroType: String
    let isAvailable = true
    
    static var typeDisplayRepresentation: TypeDisplayRepresentation = "Character"
    static var defaultQuery = CharacterQuery()
    
    // when selected
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: "\(self.avatar) \(self.id)")
    }
    
    
    static let allCharacters: [CharacterDetail] = [
        CharacterDetail(id: "Power Panda", avatar: "🐼", healthLevel: 0.14, heroType: "Forest Dweller"),
        CharacterDetail(id: "Unipony", avatar: "🦄", healthLevel: 0.67, heroType: "Free Rangers"),
        CharacterDetail(id: "Spouty", avatar: "🐳", healthLevel: 0.83, heroType: "Deep Sea Goer")
    ]
}

struct CharacterQuery: EntityQuery {
    func entities(for identifiers: [CharacterDetail.ID]) async throws -> [CharacterDetail] {
        CharacterDetail.allCharacters.filter { identifiers.contains($0.id) }
    }
    
    func suggestedEntities() async throws -> [CharacterDetail] {
        CharacterDetail.allCharacters.filter { $0.isAvailable }
    }
    
    func defaultResult() async -> CharacterDetail? {
        try? await suggestedEntities().first
    }
}

```

```swift
import WidgetKit
import SwiftUI

struct Provider: AppIntentTimelineProvider {
    func placeholder(in context: Context) -> CharacterDetailEntry {
        CharacterDetailEntry(date: Date(), detail: ConfigurationAppIntent(character: CharacterDetail.allCharacters.first!).character)
    }

    func snapshot(for configuration: ConfigurationAppIntent, in context: Context) async -> CharacterDetailEntry {
        CharacterDetailEntry(date: Date(), detail: configuration.character)
    }
    
    func timeline(for configuration: ConfigurationAppIntent, in context: Context) async -> Timeline<CharacterDetailEntry> {
        // Create the timeline and return it. The .never reload policy indicates
        // that the containing app will use WidgetCenter methods to reload the
        // widget's timeline when the details change.
        let entry = CharacterDetailEntry(date: Date(), detail: configuration.character)
        let timeline = Timeline(entries: [entry], policy: .never)
        return timeline
    }
}

struct CharacterDetailEntry: TimelineEntry {
    let date: Date
    let detail: CharacterDetail
}

struct IntentEntryView : View {
    var entry: Provider.Entry

    var body: some View {
        VStack {
            Text("Time:")
            Text(entry.date, style: .time)

            Text("Favorite Emoji:")
            Text(entry.detail.avatar)
        }
    }
}

struct Intent: Widget {
    let kind: String = "Intent"

    var body: some WidgetConfiguration {
        AppIntentConfiguration(kind: kind, intent: ConfigurationAppIntent.self, provider: Provider()) { entry in
            IntentEntryView(entry: entry)
                .containerBackground(.fill.tertiary, for: .widget)
        }
    }
}


```
