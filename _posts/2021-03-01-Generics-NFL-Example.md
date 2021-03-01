---
title: "Generics Example - 2020 NFL Season"
date: 2021-03-01
tags: [swift, iOS, Generics]
excerpt: "Exploring Flexible Swift code using the 2020 NFL Season"
---

[Link to repo with finished project](https://github.com/wdesimini/Generics_NFL_Example)

This is a tutorial on how to use generics in Swift, using fines that occurred during 2020 NFL season as an example.

Before we jump into understanding generics, let's add some starting types.

First let's add a base class `Person` and subclass `Player`, as well as a class `Team`

```swift
class Person {
    let name: String
    
    init(name: String) {
        self.name = name
    }
}

class Player: Person {
    var teamId: UUID
    
    init(name: String, teamId: UUID) {
        self.teamId = teamId
        super.init(name: name)
    }
}

class Team {
    let teamId: UUID
    var name: String
    var location: String
    
    init(teamId: UUID, name: String, location: String) {
        self.teamId = teamId
        self.name = name
        self.location = location
    }
}
```

Next, we can add an `Action` type to evaluate for fines. We'll assign the performer property the `Player` type for now.

```swift
struct Action {
    enum ActionType {
        case hitOpposingPlayerInHead
    }
    
    let type: ActionType
    let performer: Player
}
```

Lastly, we need a league to implement fines. Let's add a `League` protocol, and a struct `NFL` that conforms to our `League` protocol.

```swift
protocol League {
    func handle(action: Action)
}

struct NFL: League {
    func handle(action: Action) {
        if shouldFine(action: action) {
            fine(action: action)
        }
    }
    
    private func shouldFine(action: Action) -> Bool {
        switch action.type {
        case .hitOpposingPlayerInHead:
            return true
        }
    }
    
    private func fine(action: Action) {
        print("\(action.performer.name) has been fined")
    }
}
```

Now let's test our code to make sure it works before moving on. 

Run the following code and see what the output is.

```swift
var nfl = NFL()

let saints = Team(teamId: UUID(), name: "Saints", location: "New Orleans")
let chiefs = Team(teamId: UUID(), name: "Chiefs", location: "Kansas City")

let cam = Player(name: "Cam Jordan", teamId: saints.teamId)
let chris = Player(name: "Chris Jones", teamId: chiefs.teamId)

let camHit = Action(type: .hitOpposingPlayerInHead, performer: cam)
let chrisHit = Action(type: .hitOpposingPlayerInHead, performer: chris)

nfl.handle(action: camHit) // Cam Jordan has been fined
nfl.handle(action: chrisHit) // Chris Jones has been fined
```

Looks like our model is inaccurate. During the 2020 season Cam Jordan was fined $10,500 for hitting a Chiefs lineman in the head, but Chris Jones was not fined for any of the head punches he performed.

Let's update our `NFL` type to the following

```swift
struct NFL: League {
    var favoriteTeamIds = Set<UUID>()
    
    func handle(action: Action) {
        guard !favoriteTeamIds.contains(action.performer.teamId) else {
            return
        }
        
        if shouldFine(action: action) {
            fine(action: action)
        }
    }
    
    private func shouldFine(action: Action) -> Bool {
        switch action.type {
        case .hitOpposingPlayerInHead:
            return true
        }
    }
    
    private func fine(action: Action) {
        print("\(action.performer.name) has been fined")
    }
}
```

and update our test code and run it again

```swift
var nfl = NFL()

let saints = Team(teamId: UUID(), name: "Saints", location: "New Orleans")
let chiefs = Team(teamId: UUID(), name: "Chiefs", location: "Kansas City")

let cam = Player(name: "Cam Jordan", teamId: saints.teamId)
let chris = Player(name: "Chris Jones", teamId: chiefs.teamId)

let camHit = Action(type: .hitOpposingPlayerInHead, performer: cam)
let chrisHit = Action(type: .hitOpposingPlayerInHead, performer: chris)

nfl.favoriteTeamIds.insert(chiefs.teamId)

nfl.handle(action: camHit) // Cam Jordan has been fined
nfl.handle(action: chrisHit) // (no output)
```

Perfect, now it looks like our model's more accurate.

Now, we need to make updates so that leagues can fine teams as well. To do this, we need to be able to be flexible with what types of things we can fine.

First off we need to specify what's necessary to our league fining something. In order to do this, add protocol `Finable` that will make sure whatever we're fining will have a name and belong to a team.

```swift
protocol Finable {
    var name: String { get }
    var teamId: UUID { get }
}
```

Now we conform `Player` and `Team` to our new `Finable` protocol

```swift
class Player: Person, Finable {
    var teamId: UUID
    
    init(name: String, teamId: UUID) {
        self.teamId = teamId
        super.init(name: name)
    }
}

class Team: Finable {
    let teamId: UUID
    var name: String
    var location: String
    
    init(teamId: UUID, name: String, location: String) {
        self.teamId = teamId
        self.name = name
        self.location = location
    }
}
```

Next, let's update `Action` to a generic struct so that the `performer` property only needs to conform to the `Finable` protocol.

```swift
struct Action<PerformerType: Finable> {
    enum ActionType {
        case hitOpposingPlayerInHead
    }
    
    let type: ActionType
    let performer: PerformerType
}
```

Now we need to update our `League` models to handle the generic `Action` type. To do this, we implement a generic action handler method.

```swift
protocol League {
    func handle<T>(action: Action<T>)
}

struct NFL: League {
    var favoriteTeamIds = Set<UUID>()
    
    func handle<T>(action: Action<T>) {
        guard !favoriteTeamIds.contains(action.performer.teamId) else {
            return
        }
        
        if shouldFine(action: action) {
            fine(action: action)
        }
    }
    
    private func shouldFine<T>(action: Action<T>) -> Bool {
        switch action.type {
        case .hitOpposingPlayerInHead:
            return true
        }
    }
    
    private func fine<T>(action: Action<T>) {
        print("\(action.performer.name) has been fined")
    }
}
```

With our new generic approach to action handling and fines, we can fine teams using the same logic as what's used to fine players.

Let's try an example to see how things work

Let's add `breakCovidProtocol` to our `ActionType` enum and update `switch` statements iterating over the cases.

```swift
enum ActionType {
    case hitOpposingPlayerInHead
    case breakCovidProtocol
}

private func shouldFine<T>(action: Action<T>) -> Bool {
    switch action.type {
    case .hitOpposingPlayerInHead:
        return true
    case .breakCovidProtocol:
        return true
    }
}
```

Now run the following code to test our new generic implementation

```swift
var nfl = NFL()

let saints = Team(teamId: UUID(), name: "Saints", location: "New Orleans")
let ravens = Team(teamId: UUID(), name: "Ravens", location: "Baltimore")

nfl.favoriteTeamIds.insert(ravens.teamId)

let lockerroomCelebration = Action(type: .breakCovidProtocol, performer: saints)
let causeOutbreak = Action(type: .breakCovidProtocol, performer: ravens)

nfl.handle(action: lockerroomCelebration) // Saints have been fined
nfl.handle(action: causeOutbreak) // (no output)
```

Looks like our model's still accurate, given the Saints are the only one's getting fined in each situation.
