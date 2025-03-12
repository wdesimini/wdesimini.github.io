---
title: "How to Write a Swift Package for an API"
date: 2025-03-12
tags: [swift, iOS, api, http, testing, async]
excerpt: "A step-by-step guide to writing a Swift package for an API, using DogAPI as an example."
---

When working with APIs in Swift, creating a dedicated Swift package can help modularize your code, making it reusable and testable. 

This post details the process I use to extend a Swift package to support an additional API request, using the [Dog CEO API](https://dog.ceo/dog-api/) as an example. If you'd like to see a full implementation, you can visit my [DogAPI repo](https://github.com/wdesimini/DogAPI) on GitHub.

## 1. Add Request to Public API Client

Start by adding an empty request method in your public interface that returns a value in the desired format:

```swift
import Foundation

public struct DogAPI {
    private let client: DogAPIClient

    public init(session: URLSession = .shared) {
        self.client = .init(session: session)
    }

    /// Fetch single random image from all dogs collection
    /// - Returns: URL of random dog image
    public func fetchRandomImage() async throws -> URL {
        fatalError("implement")
    }
}
```

## 2. Add a Unit Test for the Request

Before implementing the request, I usually like to write a unit test that outlines the request I'm trying to send and response I'm expecting. This way I can verify my request implementation:
- Hits the correct URL and endpoint.
- Handles the expected response correctly. 
- Parses the expected data format correctly.

Create a test in `DogAPITests` that passes the expected data of a successful response for the request URL into some mock `URLProtocol`, and verifies the returned value of the new request matches the expected output:

*Note: In this example, `MockURLProtocol` and `URLSession.mock` are preexisting utilities used to stub network requests. See [DogAPI repo](https://github.com/wdesimini/DogAPI) for further details.*

```swift
import XCTest
@testable import DogAPI

final class DogAPITests: XCTestCase {
    private var api: DogAPI!

    override func setUp() {
        api = DogAPI(session: .mock)
    }

    override func tearDown() {
        MockURLProtocol.reset()
        api = nil
    }

    func test_fetchRandomImage() async throws {
        // given random image request will succeed
        let url = URL(string: "https://dog.ceo/api/breeds/image/random")!
        let expectedResponse = HTTPURLResponse(url: url, statusCode: 200, httpVersion: nil, headerFields: nil)
        let expectedData = #"{"message":"https://images.dog.ceo/breeds/pembroke/n02113023_219.jpg","status":"success"}"#.data(using: .utf8)!
        MockURLProtocol.expect(.init(data: expectedData, response: expectedResponse), for: url)
        // when user requests random image from all dogs
        let randomImage = try await api.fetchRandomImage()
        // then random image is returned
        XCTAssertEqual(randomImage, URL(string: "https://images.dog.ceo/breeds/pembroke/n02113023_219.jpg"))
    }
}
```

## 3. Define the Endpoint

Extend the `DogAPIEndpoint` enum to include an endpoint for the new request:

```swift
enum DogAPIEndpoint {
    case randomImage
    
    var path: String {
        switch self {
        case .randomImage:
            "breeds/image/random"
        }
    }
}
```

## 4. Implement the Request in the Public API Client

Go back to the `DogAPI` and modify the method added earlier to fetch from the new endpoint using the `DogAPIClient`:

*Note: In this example, `DogAPIClient` receives the endpoint, builds the URL from the endpoint, executes the request using a given `URLSession`, and parses the response. See [DogAPI repo](https://github.com/wdesimini/DogAPI) for further details and implementation.*

```swift
import Foundation

public struct DogAPI {
    private let client: DogAPIClient

    public init(session: URLSession = .shared) {
        self.client = .init(session: session)
    }

    /// Fetch single random image from all dogs collection
    /// - Returns: URL of random dog image
    public func fetchRandomImage() async throws -> URL {
        try await client.fetch(.randomImage)
    }
}
```

## 5. Run the Test and Verify

Now, run the test added in Step 2. If it passes, your implementation is correct!

---

By following these steps, you’ve successfully added an API request to a Swift package while ensuring it is well-tested and modular. 🎉