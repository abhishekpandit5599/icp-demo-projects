## Demo Projects on ICP

### File Storing System in ICP
`main.mo`
```js
import Text "mo:base/Text";
import Error "mo:base/Error";
import HashMap "mo:base/HashMap";
import Iter "mo:base/Iter";
actor {
  type ResponseData = {
    status : Bool;
    data : Text;
  };
  type ResponseDataAllFiles = {
    status : Bool;
    data : [(Text,Text)];
  };

  var fileHashMap = HashMap.HashMap<Text, Text>(0, Text.equal, Text.hash);
  stable var entries : [(Text, Text)] = [];

  public func saveFile(uuid : Text, file : Text) : async ResponseData {
    fileHashMap.put(uuid, file);
    return {
      status = true;
      data = file;
    };
  };

  public query func getFile(uuid : Text) : async ResponseData {
    var file : Text = switch (fileHashMap.get(uuid)) {
      case (null) { throw Error.reject("File not exist") };
      case (?result) { result };
    };
    return {
      status = true;
      data = file;
    };
  };

  public query func getAllFiles() : async ResponseDataAllFiles {
    let files = Iter.toArray<(Text, Text)>(fileHashMap.entries());
    return {
      status = true;
      data = files;
    }
  };

  system func preupgrade() {
    entries := Iter.toArray(fileHashMap.entries());
  };

  system func postupgrade() {
    fileHashMap := HashMap.fromIter<Text, Text>(entries.vals(), 1, Text.equal, Text.hash);
    entries := [];
  };
};
```


### ToDo with login in motoko
`main.mo`
```js
import HashMap "mo:base/HashMap";
import Bool "mo:base/Bool";
import Text "mo:base/Text";
import Option "mo:base/Option";
import Array "mo:base/Array";
import Debug "mo:base/Debug";
import Buffer "mo:base/Buffer";
import Error "mo:base/Error";
import List "mo:base/List";
import Hash "mo:base/Hash";

actor UserRegistration {
  // Creating two arrays to store pending and completed tasks
  var pendingTodo : [Text] = [];
  var completedTodo : [Text] = [];

  type User = {
    username : Text;
    pass : Text;
    pendingTodo : [Text];
    completedTodo : [Text];
  };

  // create hashmap to store data with every uniquer username
  let usersByUsername = HashMap.HashMap<Text, User>(0, Text.equal, Text.hash);
  var currentUser : Text = "";

  // Function to check if a username is taken
  public query func isUsernameTaken(username : Text) : async Bool {
    let value : Bool = switch (usersByUsername.get(username)) {
      case (null) {
        return false;
      };
      case _ {
        return true;
      };
    };
    return value;
  };

  // this function will return the password from backend to frontend
  public func isCorrectPassword(username : Text, password: Text) : async Text {
    switch (usersByUsername.get(username)) {
      case (null) {
        return "User doesn't exist";
      };
      case (?result) {
        if(result.pass == password) {
          currentUser := username;
          return "Success";
        };
        return "Username or Password is incorrent";
      };
    };
  };

  // Function to register a new user
  public func registerUser(users : Text, password : Text) : async Bool {
    let value : Bool = switch (usersByUsername.get(users)) {
      case (null) {
        let newuser : User = {
          username = users;
          pass = password;
          pendingTodo = [];
          completedTodo = [];
        };
        usersByUsername.put(users, newuser);
        return true;
      };
      case _ {
        return false;
      };
    };
    return value;
  };


  public func getCurrentUser() : async Text {
    return currentUser;
  };

  // function to add task in pending list
  public func addinPendingList(value : Text) : async () {
    switch (usersByUsername.get(currentUser)) {
      case (?user) {
        let updatedPendingTodo = Array.append(user.pendingTodo, [value]);
        usersByUsername.put(currentUser, { user with pendingTodo = updatedPendingTodo });
      };
      case (null) {
        Debug.print("User not find");
      };
    };
  };

  // function to fetch pending todo list
  public func getPendingtodo() : async [Text] {
    switch (usersByUsername.get(currentUser)) {
      case (?user) {
        return user.pendingTodo;
      };
      case (null) {
        return [];
      };
    };
  };

  // function to add task in completed list but at first remove the task from pending task list
  public func addinCompletedList(value : Text) : async () {
    switch (usersByUsername.get(currentUser)) {
      case (?user) {
        let pendingTasks = user.pendingTodo;
        let completedTasks = user.completedTodo;
        let newPendingTasks = Buffer.Buffer<Text>(pendingTasks.size() -1);

        for (i in pendingTasks.vals()) {
          if (i == value) {

          } else {
            newPendingTasks.add(i);
          };
        };
        // Remove the task from pendingTodo
        let updatedPendingTodo = newPendingTasks.toArray();
        // Add the task to completedTodo
        let updatedCompletedTodo = Array.append(completedTasks, [value]);
        usersByUsername.put(
          currentUser,
          {
            user with
            pendingTodo = updatedPendingTodo;
            completedTodo = updatedCompletedTodo;
          },
        );
      };
      case (null) {
        Debug.print("User not find");
      };
    };
  };

  // function to fetch completed task list
  public func getCompletedtodo() : async [Text] {
    switch (usersByUsername.get(currentUser)) {
      case (?user) {
        return user.completedTodo;
      };
      case (null) {
        return [];
      };
    };
  };
};
```


### Simple TODO  without Login 
```js
import Map "mo:base/HashMap";
import Hash "mo:base/Hash";
import Nat "mo:base/Nat";
import Iter "mo:base/Iter";
import Text "mo:base/Text";

// Define the actor
actor Assistant {

  type ToDo = {
    description: Text;
    completed: Bool;
  };

  func natHash(n : Nat) : Hash.Hash { 
    Text.hash(Nat.toText(n))
  };

  var todos = Map.HashMap<Nat, ToDo>(0, Nat.equal, natHash);
  var nextId : Nat = 0;

  public query func getTodos() : async [ToDo] {
    Iter.toArray(todos.vals());
  };

  // Returns the ID that was given to the ToDo item
  public func addTodo(description : Text) : async Nat {
    let id = nextId;
    todos.put(id, { description = description; completed = false });
    nextId += 1;
    id
  };

  public func completeTodo(id : Nat) : async () {
    ignore do ? {
      let description = todos.get(id)!.description;
      todos.put(id, { description; completed = true });
    }
  };

  public query func showTodos() : async Text {
    var output : Text = "\n___TO-DOs___";
    for (todo : ToDo in todos.vals()) {
      output #= "\n" # todo.description;
      if (todo.completed) { output #= " ✔"; };
    };
    output # "\n"
  };

  public func clearCompleted() : async () {
    todos := Map.mapFilter<Nat, ToDo, ToDo>(todos, Nat.equal, natHash, 
              func(_, todo) { if (todo.completed) null else ?todo });
  };
}
```

### Guest Book
```js
import List "mo:base/List";
import Option "mo:base/Option";
import Nat32 "mo:base/Nat32";
import Trie "mo:base/Trie";
import Time "mo:base/Time";
import Cycles "mo:base/ExperimentalCycles";

actor {
  public type BlogId = Nat32;

  public type Blog = {
    body : Text;
    author : Text;
    timestamp : Int;
  };

  private stable var next : BlogId = 0;

  private stable var praises : Trie.Trie<BlogId, Nat32> = Trie.empty();

  private stable var poops : Trie.Trie<BlogId, Nat32> = Trie.empty();

  // The blogs data store.
  private stable var blogs : Trie.Trie<BlogId, Blog> = Trie.empty();

   // Create a blog post.
  public func create(blog : Blog) : async BlogId {
    let blogId = next;
    next += 1;

    if (blog.author.size() > 256) {
      // TODO: Make this return an error.
      return 0
    };

    blogs := Trie.replace(
      blogs,
      key(blogId),
      eq,
      ?blog,
    ).0;
    return blogId;
  };

  public func createPraise(blogId: BlogId) : async BlogId {
    var praisesForTheBlog = Trie.find(praises, key(blogId), eq);
    if (Option.isNull(praisesForTheBlog)) {
      praisesForTheBlog := Option.make<Nat32>(0);
    };

    praisesForTheBlog := Option.make<Nat32>(Nat32.add(Option.unwrap(praisesForTheBlog), 1));
    praises := Trie.replace(
      praises,
      key(blogId),
      eq,
      praisesForTheBlog
    ).0;
    return blogId;
  };

  public func createPoop(blogId: BlogId) : async BlogId {
    var poopsForTheBlog = Trie.find(praises, key(blogId), eq);
    if (Option.isNull(poopsForTheBlog)) {
      poopsForTheBlog := Option.make<Nat32>(0);
    };

    poopsForTheBlog := Option.make<Nat32>(Nat32.add(Option.unwrap(poopsForTheBlog), 1));
    poops := Trie.replace(
      poops,
      key(blogId),
      eq,
      poopsForTheBlog
    ).0;
    return blogId;
  };

  public query func read(blogId : BlogId) : async ?Blog {
    let result = Trie.find(blogs, key(blogId), eq);
    return result;
  };

  public query func readAll() : async Trie.Trie<BlogId, Blog>  {
    return blogs;
  };

  public query func readAllPraises() : async Trie.Trie<BlogId, Nat32>  {
    return praises;
  };

  public query func readAllPoops() : async Trie.Trie<BlogId, Nat32>  {
    return poops;
  };

   private func eq(x : BlogId, y : BlogId) : Bool {
    return x == y;
  };

  private func key(x : BlogId) : Trie.Key<BlogId> {
    return { hash = x; key = x };
  };

  //Internal cycle management
  public func acceptCycles() : async () {
    let available = Cycles.available();
    let accepted = Cycles.accept(available);
    assert (accepted == available);
  };

  public query func availableCycles() : async Nat {
    return Cycles.balance();
  };
};
```
### Blockchain-based Certification System
`main.mo`
```js
import HashMap "mo:base/HashMap";
import Principal "mo:base/Principal";
import List "mo:base/List";
import Iter "mo:base/Iter";
import Time "mo:base/Time";
import Debug "mo:base/Debug";

actor certification {

  private stable var registeredEntries : [(Principal, User)] = [];

  // private stable var savedList : [Principal] = [];

  type HashMap<K, V> = HashMap.HashMap<K, V>;

  stable var registedOnesList : List.List<Principal> = List.nil<Principal>();

  var registedUsersHashmap : HashMap<Principal, User> = HashMap.HashMap<Principal, User>(5, Principal.equal, Principal.hash);

  let certificateHolders : HashMap<Principal, Certification> = HashMap.HashMap<Principal, Certification>(5, Principal.equal, Principal.hash);

  // Registeration
  type User = {
    name : Text;
    isRegistered : Bool;
  };

  type Certification = {
    name : Text;
    issueTimestamp : Int;
    isRevoked : Bool;
  };

  public shared (msg) func registerUser(namee : Text) : async Text {

    let obj : User = { name = namee; isRegistered = true };

    let userId : Principal = msg.caller;

    var iter = Iter.fromList(registedOnesList);

    for (user in iter) {
      if (user == userId) {
        return "you are already registered buddy!";
      };
    };

    registedOnesList := Iter.toList(iter);

    registedUsersHashmap.put(userId, obj);
    registedOnesList := List.push(userId, registedOnesList);

    return "success registering!!";

  };

  public func issueCertificate(address : Principal, certiName : Text) : async Text {

    var iter = Iter.fromList(registedOnesList);
    var userFound : Bool = false;

    for (user in iter) {
      if (user == address) {

        let obj : Certification = {
          name = certiName;
          issueTimestamp = Time.now();
          isRevoked = false;
        };

        certificateHolders.put(address, obj);

        Debug.print("Certificate has been issued!");
        userFound := true;
        // return "you are already registered buddy!";
      } 
    };

    if(userFound == false){
      return "user has not registered himself";
    };

    registedOnesList := Iter.toList(iter);

    return "Certificate has been issued!";

  };

  public shared(msg) func getId(): async Principal{
    return msg.caller;
  };

  public query func getUsers() : async List.List<Principal> {
    return registedOnesList;
  };

  system func preupgrade() {
    registeredEntries := Iter.toArray(registedUsersHashmap.entries());
  };

  system func postupgrade() {

    registedUsersHashmap := HashMap.fromIter<Principal, User>(registeredEntries.vals(), 1, Principal.equal, Principal.hash);

    };

};

```


<<<<<<< Updated upstream
## Mobile-Calcy

`main.mo`

```js
import Float "mo:base/Float";
actor Calculator {

  var num : Float = 0;

  public func add(val1 : Float, val2 : Float) : async Float {
    num := val1 +val2;
    return num;
  };

  public func subtract(val1 : Float, val2 : Float) : async Float {
    num := val1 -val2;
    return num;
  };

  public func multiply(val1 : Float, val2 : Float) : async Float {
    num := val1 * val2;
    return num;
  };

  public func divison(val1 : Float, val2 : Float) : async ?Float {
    if (val2 == 0) {
      return null;
    } else {
      num := val1 / val2;
      return ?num;
    };
  };

  public func clearAll() : async Float {
    num := 0;
    return num;
  };

};
```

`App.jsx`

```js
import React, { useState } from "react";
import { calcy_backend } from "../../../declarations/calcy_backend";

function App() {
  const [curr, setCurr] = useState("");
  const [prev, setPrev] = useState("");
  const [operand, setOperand] = useState("");

  const numHandler = (e) => {
    if (curr.includes(".") && e.target.value === ".") return;
    curr ? setCurr((prev) => prev + e.target.value) : setCurr(e.target.value);
  };

  const clearHandler = async () => {
    await calcy_backend.clearAll();
    setCurr("");
    setPrev("");
    setOperand("");
  };
  const clearLastHandler = () => {
    if (operand) return;
    let res = curr.slice(0, -1);
    setCurr(res);
  };

  const operandHandler = (e) => {
    setOperand(e.target.value);
    if (curr === "") return;
    if (prev !== "") calculate();
    else {
      setPrev(curr);
      setCurr("");
    }
  };
  const calculate = async () => {
    let cal, val1, val2;
    switch (operand) {
      case "+":
        val1 = Number(prev);
        val2 = Number(curr);
        cal = String(await calcy_backend.add(val1, val2));
        break;

      case "-":
        val1 = Number(prev);
        val2 = Number(curr);
        cal = String(await calcy_backend.subtract(val1, val2));
        break;

      case "*":
        val1 = Number(prev);
        val2 = Number(curr);
        cal = String(await calcy_backend.multiply(val1, val2));
        break;

      case "/":
        val1 = Number(prev);
        val2 = Number(curr);
        cal = String(await calcy_backend.divison(val1, val2));
        break;
      default:
        return;
    }
    console.log("cal", cal);

    setPrev(cal);
    setCurr("");
  };

  let numArray = [];
  for (let i = 0; i <= 9; i++) {
    numArray.push(i);
  }
  numArray.push(".");

  const operandsss = ["+", "-", "*", "/"];

  return (
    <div className="App">
      <h1>calculator</h1>
      <div className="screen">
        <span>{prev}</span>
        <span>{operand}</span>
        <span>{curr}</span>
      </div>
      {operandsss.map((opr, index) => (
        <button
          key={index}
          value={opr}
          className="button opr"
          onClick={operandHandler}
        >
          {opr}
        </button>
      ))}
      {numArray.map((num, index) => (
        <button key={index} value={num} className="button" onClick={numHandler}>
          {num}
        </button>
      ))}
      <input
        type="button"
        value="X"
        className="button btn2"
        onClick={clearLastHandler}
      />
      <input
        type="button"
        value="AC"
        className="button btn"
        onClick={clearHandler}
      />
      <input
        type="button"
        value="="
        className="button button1"
        onClick={calculate}
      />
    </div>
  );
}

export default App;
```
`main.css`

```js
body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
    "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

code {
  font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
    monospace;
}

.App {
  width: 390px;
  margin: 100px auto;
  align-items: center;
}

.screen {
  width: 300px;
  height: 80px;
  text-align: right;
  border: 3px solid rgb(88, 87, 87);
  font-size: 30px;
  padding: 10px;
  border-radius: 10px;
  background-color: white;
}

.button {
  width: 80px;
  height: 80px;
  background-color: black;
  color: white;
  border-radius: 10px;
  font-size: 25px;
}

body {
  background-color: lightgray;
}

.btn {
  width: 160px;
  background-color: white;
  color: black;
}

.opr {
  background-color: rgb(158, 55, 7);
}

.btn2 {
  background-color: rgb(65, 63, 63);
}

.button1 {
  width: 160px;
  background-color: yellowgreen;
}
```

`package.json`
```js
// change your depedencies with this in package.json

"dependencies": {
    "@dfinity/agent": "^0.19.3",
    "@dfinity/candid": "^0.19.3",
    "@dfinity/principal": "^0.19.3",
    "@emotion/react": "^11.11.1",
    "@emotion/styled": "^11.11.0",
    "@material-ui/core": "^4.12.4",
    "@material-ui/icons": "^4.11.3",
    "@mui/icons-material": "^5.14.14",
    "@mui/material": "^5.14.14",
    "autoprefixer": "^10.4.16",
    "postcss": "^8.4.31",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "ts-loader": "^9.5.0"
  },
  ```
