*Separation of concerns*: Each module in an app should do exactly **one** thing.

- Control: a reusable GUI element
- Widget: an embeddable mini app, e.g. managing a calendar, a map, third party services, etc
- Layer: logical grouping of functionality. For example, a data layer encapsulates state, a presentation layer handles display and UI concerns
- tier: runtime environments layer are deployed on. Possible to run multiple layers on a tier, however tiers should be kept independent enough to run on separate environments or machines.

#####Client-side concerns



