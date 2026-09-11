# GymOps — UX Design Principles

> Placeholder. To be expanded.

To bridge the gap between a data model (Entity Diagram) and user behavior (Use Case List) without introducing "UX fluff," you must rely on Object-Oriented UX (OOUX) and Information Density Maps.
The goal is to prevent the user's brain from having to mentally reconstruct the database structure. Instead, the interface should map directly to the mental models established by the entities themselves.
Popularized by Sophia Prater, OOUX is a strict, no-nonsense methodology that directly maps database entities into modular user interface objects before you ever think about a layout. This eliminates the cognitive load of users guessing where a piece of data lives.
Follow this strict mathematical 4-step translation pipeline:

   1. Extract Objects (from your Entity Diagram): Every major entity (e.g., User, Invoice, Project) becomes a concrete visual container (a "Card" or a "Workspace").
   2. Map Relationships (from your Entity Diagram): If a Project has many Invoices, the Project interface must inherently contain a nested list of Invoices. No forcing the user to navigate to an isolated "Invoices" tab and filter by project.
   3. Layer Actions (from your Use Case List): Map every use case directly as an explicit action button inside the object container it modifies. If the use case is "Approve Invoice," that button lives directly on the Invoice card, not in a generic top-level settings menu.
   4. Prioritize Attributes (Tufte's Data-Ink): Sort the fields of your entity from highest to lowest utility based on the use cases. Hide secondary attributes behind a progressive disclosure layer (e.g., a "Show Details" chevron) to protect the user's foveal vision from information overload.

To ensure a maximum coherence screen-flow, map your entities and use cases into this structural matrix to determine exactly how screens transition.

| From Entity (Context) | Use Case / Action | Resulting State / Screen | Cognitive Flow Principle |
|---|---|---|---|
| Parent Object (e.g., Project) | View Details | Nested Child List (e.g., Invoices) | Spatial Anchoring: Keep the user in the parent environment; do not jump contexts. |
| Child Object (e.g., Invoice) | Trigger State Change (e.g., Approve) | Inline Mutation (State updates in place) | Direct Manipulation: Minimize page refreshes or routing steps; the object updates where it sits. |
| Any Object | Cross-Reference (e.g., click Assignee) | Pivot Link (Seamlessly transition to User profile) | Information Continuity: The user follows a relational thread without resetting their mental search. |

When designing the screen flow, use this strict structural checklist to minimize cognitive friction:

* 
* Law of Proximity (Gestalt): Group relational data fields together. If an invoice Status is dependent on the Due Date, those two strings of text must be visually locked together in space.
* Eliminate State Discontinuity: When a user clicks an entity to view its details, the entity's core visual anchor (e.g., its title or unique ID) must remain in the exact same screen position or transition smoothly. Moving a title from the left side of a list to the center of a detail view forces a cognitive eye-re-scan.
* Enforce a Single Primary Action: For every node in your screen-flow, there must only be one high-contrast visual call-to-action (the dominant use case for that screen state). Secondary use cases must use lower luminance contrast weights.
* 


## Appendix

- https://m3.material.io/
