# How to make an API Request in SwiftUI with MVVM pattern
![image](https://user-images.githubusercontent.com/40492085/211134190-aea91ee7-99c6-4a7a-aa27-2451d4addcc0.png)
## API Request 📩
An API works by requesting information from a server and then receiving a response from it. Whenever you make a call to a server using an API, this counts as an API request. Some of the operations that are considered to be API requests include logins, queries, and saves, among others.

Here we will learn about implementing Login API request to authenticate a user, using Alamofire with MVVM pattern.

MVVM pattern 📃
![image](https://user-images.githubusercontent.com/40492085/211134410-e3973604-927d-417a-b9c5-dd3c84f33567.png)

Model-View-ViewModel (MVVM) is structured to separate program logic and user interface controls.

## Alamofire 🐦
Alamofire is an HTTP network-based library used to handle the web request and response in iOS and macOS, which is added as an external dependency. It provides the request and response methods, JSON parameter and response serialization, authentication, and many other features.

Enough theory! It's a long way to go… let us dive into the code! 💻

⏬ Below is a brief diagram of the Project setup we will be following…
![image](https://user-images.githubusercontent.com/40492085/211134884-554c78ae-326a-41b5-a317-2d15ef33dad8.png)

**1. Models**

Codable: A type that can convert itself into and out of an external representation. Encodable and decodable data types provide compatibility with external representations such as JSON.️

Email and password for request and token property in response model to set the JSON data, which we are going to get from our API response.

Here we will use Login API from the site: https://reqres.in/

```
//Request
struct CreateLoginRequest: Codable {
  var email: String?
  var password: String?
}
//Response
struct CreateLoginResponse: Codable {
  var token: String
}
```

**2. API Manager Class**

A shared Alamofire request class, with common required request parameters such as serverURL, parameters, headers, and completion handlers for handling responses.

For the API request, we can simply use Alamofire request method by using their pre-built enums.

> As Alamofire is an external dependency, we will need to add import Alamofire statement in project files and pod ‘Alamofire’ in project’s PodFile.

⏬ Below is the final class…

```
import Alamofire
public typealias FailureMessage = String

public class APIManager {
  public static let shared = APIManager()
  
  func callAPI(serverURL: String, method: HTTPMethod = .get, headers: HTTPHeaders? = nil, parameters: Parameters? = nil, success: @escaping ((AFDataResponse<Any>) -> Void), failure: @escaping ((FailureMessage) -> Void)) {
    
    guard var url = URLComponents(string: "\(serverURL)") else {
      failure("Invalid URL")
      return
    }
    // Network request
    AF.request(url, method: method, parameters: parameters, encoding: JSONEncoding.default, headers: headers)
      .responseJSON { response in
        switch response.result {
        case .success:
          success(response)
        case let .failure(error):
          failure(error.localizedDescription)
        }
      }
  }
}
```

**3. API Services**

A separate struct for making a Login API request using Alamofire shared class for user authentication.

From the request it will return an API success response or a failure response by decoding the data being returned from the API to the `ViewModel`, using `JSONDecoder`.

⏬ Below is the final struct…

```
import Alamofire
struct APIServices {
  public static let shared = APIServices()
  func callCreateLogin(parameters: Parameters? = nil, success: @escaping (_ result: CreateLoginResponse?) -> Void, failure: @escaping (_ failureMsg: FailureMessage) -> Void) {
    var headers = HTTPHeaders()
    headers["content-type"] = "application/json"
    APIManager.shared.callAPI(serverURL: "https://reqres.in/api/login", method: .post, headers: headers, parameters: parameters, success: { response in
      do {
        if let data = response.data {
          let createLoginResponse = try JSONDecoder().decode(CreateLoginResponse.self, from: data)
          success(createLoginResponse)
        }
      } catch {
        failure(FailureMessage(error.localizedDescription))
      }
    }, failure: { error in
      failure(FailureMessage(error))
    })
  }
}
```

**4. Login View**

For getting user inputs to authenticate it using email and password.

An instance for ViewModel will be declared for attaching it with the view.

```
@StateObject var loginViewModel = LoginViewModel()
```
Here we will add two TextField’s and a Login Button to authenticate the user.
```
//Email field
      TextField("Email", text: $loginViewModel.youremailText)
        .font(Font.system(size: 15))
        .padding()
        .background(
          RoundedRectangle(cornerRadius: 10, style: .continuous)
            .stroke(Color.gray.opacity(0.5), lineWidth: 3)
        ).padding()
//Password field
      SecureField("Password", text: $loginViewModel.passwordText)
        .font(Font.system(size: 15))
        .padding()
        .background(
          RoundedRectangle(cornerRadius: 10, style: .continuous)
            .stroke(Color.gray.opacity(0.5), lineWidth: 3)
        ).padding()
```
In Button action, we will make the Login request by passing required binding parameters, i.e. password and email which are attached with ViewModel.
```
//Sign In Button
      Button("SignIn", action: {
        let createLoginRequest = CreateLoginRequest(email: loginViewModel.emailField, password: loginViewModel.passwordField)
        loginViewModel.createLogin(request: createLoginRequest)
      })
    }
```
**5. Login View Model**

ObservableObject is used within a custom class or model to keep track of the state, a protocol within the Combine framework. It is used with the most powerful property wrapper @Published, used before any properties that should trigger changes for the view.

Login View Model conforms to ObservableObject and has declared publishers for binding data.
```
class LoginViewModel: ObservableObject {
  @Published var emailField: String = ""
  @Published var passwordField: String = ""
  @Published var createLoginResponse: CreateLoginResponse?
func createLogin(request: CreateLoginRequest) {
    APIServices.shared.callCreateLogin(parameters: request.dictionary ?? [:]) { response in
      if let response = response {
        print(response)
      }
    }
      failure: { error in
        print(error)
      }
  }
}
```
The request made here gets a response from the API using Services.

That's it! You are done with the code. Lets see the output!🔮
![image](https://user-images.githubusercontent.com/40492085/211138871-ae3318a4-be64-45d3-963e-f31211c30d3e.png)

As we will add the email and password, a token will be printed in the console as a response from the API, after a successful request from the SignIn button.


Response Success!!!
Note: Apart from Alamofire there are also other network libraries for API in Swift for example URL Session is also a good choice.

Thank you for making out this far! 😅

I hope you have learned about how an API request is setup in MVVM pattern for SwiftUI.
