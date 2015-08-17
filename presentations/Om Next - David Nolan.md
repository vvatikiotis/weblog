Om Next - David Nolan - https://www.youtube.com/watch?v=ByNs9TG30E8

###Problem
- How do you sync server and *remote* client?
  - Getting initial data
  - Getting changes i.e.changeset
  
###Possible solutions
- Relay/GraphQL - Facebook 
- JSONGraph/Falcor - Netflix
- Pull Syntax - Datomic

Browser is fundamentally about *documents* and todays web apps are not documents but web UI components.

REST doesn't work for remote/mobile rich client apps (bandwidth is scarce). Mobile clients needs multiple resources (endpoints). Forcing REST  to serve different clients like web/different mobile devices (display sizes and UXs) and increasingly *new* devices is problematic. Why? because bandwidth is scarce so you force into 1 endpoint different sets of data in order to save network requests, thus violating REST.

*Composition has to be maintained!*

###Synthesise ideas from the possible solutions
Use ideas for Om Next.

End of notes (for now). I don't know any ClojureScript to take any meaningful notes (I'll work on that).
