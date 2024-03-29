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

### Guest Book

```js
import List "mo:base/const inputtdl = document.querySelector('.textarea')
const buttontdl = document.querySelector('.buttoninput')
const listtdl = document.querySelector('.todolist')

function clickButton(e) {
    e.preventDefault()
    addTodo()
}

// adding todoList
function addTodo() {
    const itemall = document.createElement('div')
    itemall.classList.add('itemall')

    const item = document.createElement('p')
    item.classList.add('item')
    item.innerText = inputtdl.value
    itemall.appendChild(item)

    if (inputtdl.value === '') return

    const checkbutton = document.createElement("button")
    checkbutton.innerHTML = '<i class="fa-solid fa-check"></i>'
    checkbutton.classList.add("check-button")
    itemall.appendChild(checkbutton)

    const trashbutton = document.createElement("button")
    trashbutton.innerHTML = '<i class="fa-solid fa-trash"></i>'
    trashbutton.classList.add("trash-button")
    itemall.appendChild(trashbutton)

    listtdl.appendChild(itemall)
    inputtdl.value = ''
}

// checking and delete todoList
function okdel(e) {
    const item = e.target

    // check
    if (item.classList[0] === 'check-button') {
        const todolist = item.parentElement
        todolist.classList.toggle('checklist')
    }

    // delete
    if (item.classList[0] === 'trash-button') {
        const todolist = item.parentElement
        todolist.remove()
    }
}

buttontdl.addEventListener('click', clickButton)
listtdl.addEventListener('click', okdel)List";
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


## BMI CALCULATOR

`main.mo`

```js
actor {
    public func calculateBMI(weightInKg: Float, heightInCM: Float) : async (Float, Text) {
        let bmi = (weightInKg  / heightInCM / heightInCM)*10000;
        let category =
            if (bmi < 18.5) {
                "Underweight"
            } else if (bmi >= 18.5 and bmi < 24.9) {
                "Normal weight"
            } else if (bmi >= 24.9 and bmi < 29.9) {
                "Overweight"
            } else {
                "Obesity"
            };

        return (bmi, category);
    }

};

```

`main.css`

```js

/* Import Google Font - Poppins */
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap');
    *{
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: 'Poppins', sans-serif;
    }

    body{
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    background: rgb(97,138,254,0.9);
    }
/*
  .container {
    min-width: 250px;
  } */

  .box {
    width: 500px;
    background: #fafafa;
    border-radius: 12px;
    text-align: center;
    box-shadow: 2px 2px 20px 5px rgba(0,0,0,0.5);

  }


  h1 {
    color: rgb(0, 0, 0, 0.7);
    font-weight: bold;
    font-size: 36px;
    padding: 30px 0;

  }



    .content {
        padding: 0 30px;

    }

    .input {
        background: #fff;
        box-shadow: 0px 0px 95px -30px rgba(0, 0, 0, 0.15);
        border-radius: 12px;
        padding: 20px 0;
        margin-bottom: 20px;
    }


    .input label {
        display: block;
        font-size: 18px;
        font-weight: 600;
        color: #000;
        margin-bottom: 20px;
    }
    .input input {
        outline: none;
        border: none;
        border-bottom: 1px solid #4f7df9;
        width: 50%;
        text-align: center;
        font-size: 28px;
        font-family: "Nunito", sans-serif;
    }



    .inputW {
        background: #fff;
        box-shadow: 0px 0px 95px -30px rgba(0, 0, 0, 0.15);
        border-radius: 12px;
        padding: 10px 0;
        margin-bottom: 20px;
    }


    .inputW label {
        display: block;
        font-size: 18px;
        font-weight: 600;
        color: #000;
        margin-bottom: 20px;
    }
   .inputW input {
        outline: none;
        border: none;
        border-bottom: 1px solid #4f7df9;
        width: 50%;
        text-align: center;
        font-size: 28px;
        font-family: "Nunito", sans-serif;
    }

    .inputH {
        background: #fff;
        box-shadow: 0px 0px 95px -30px rgba(0, 0, 0, 0.15);
        border-radius: 12px;
        padding: 10px 0;
        margin-bottom: 20px;
        margin-right: 20px;
    }


    .inputH label {
        display: block;
        font-size: 18px;
        font-weight: 600;
        color: #000;
        margin-bottom: 20px;
    }
   .inputH input {
        outline: none;
        border: none;
        border-bottom: 1px solid #4f7df9;
        width: 50%;
        text-align: center;
        font-size: 28px;
        font-family: "Nunito", sans-serif;
    }


button.calculate {
    cursor: pointer;
    font-family: "Nunito", sans-serif;
    color: #fff;
    background: rgb(97,138,254,1);
    font-size: 16px;
    border-radius: 7px;
    padding: 12px 0;
    width: 100%;
    outline: none;
    border: none;
    transition: all 0.5s;
  }
  button.calculate:hover {
    background: #0044ff;
  }



.result {
    padding: 10px 20px;
  }
  .result p {
    font-weight: 600;
    font-size: 22px;
    color: #000;
    margin-bottom: 15px;
  }
  .result #result {
    font-size: 36px;
    font-weight: 900;
    color: #4f7df9;
    background-color: #eaeaea;
    display: inline-block;
    padding: 7px 20px;
    border-radius: 55px;
    margin-bottom: 25px;
  }
  #comment {
    color: #4f7df9;
    font-weight: 800;
  }

.comment {
    display: none;
    border: dashed 1px;
    border-radius: 7px;
    padding: 5px;
}

.footer {
    display: flex;
    justify-content: start;
    align-items: center;
    padding: 10px 15px
}

.footer-text {
    text-decoration: none;
    color: rgba(40, 40, 40, 0.8);
    font-family: 'Poppins', sans-serif;
    font-weight: 200;
    font-size: 14px;
    transition: all 0.5;
}

.footer-text:hover {
    color: rgba(40, 40, 40, 1);
}

.gender {
    display: flex;
    justify-content: space-around;
    align-items: center;
    align-content: center;
    background: #fff;
    box-shadow: 0px 0px 95px -30px rgba(0, 0, 0, 0.15);
    border-radius: 12px;
    padding: 20px 0;
    margin-bottom: 20px;
}

  /* Style for Radio */
.gender .container {
    display: block;
    position: relative;
    padding-left: 35px;
    /* margin-bottom: 12px; */
    margin-top: 7px;
    cursor: pointer;
    font-size: 22px;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
  }

  /* Hide the browser's default radio button */
  .gender .container input {
    position: absolute;
    opacity: 0;
    cursor: pointer;
  }

  /* Create a custom radio button */
  .checkmark {
    position: absolute;
    top: 0;
    left: 0;
    height: 25px;
    width: 25px;
    background-color: #eee;
    border-radius: 50%;
  }

  /* On mouse-over, add a grey background color */
  .gender .container:hover input ~ .checkmark {
    background-color: #ccc;
  }

  /* When the radio button is checked, add a blue background */
  .gender .container input:checked ~ .checkmark {
    background-color: #2196F3;
  }

  /* Create the indicator (the dot/circle - hidden when not checked) */
  .checkmark:after {
    content: "";
    position: absolute;
    display: none;
  }

  /* Show the indicator (dot/circle) when checked */
  .gender .container input:checked ~ .checkmark:after {
    display: block;
  }

  /* Style the indicator (dot/circle) */
  .gender .container .checkmark:after {
       top: 9px;
      left: 9px;
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: white;
  }
  /* END Style for Radio */

.containerHW {
    display: flex;
    justify-content: space-around;
    align-items: center;
}

/* The Modal (background) */
.modal {
  display: none; /* Hidden by default */
  position: fixed; /* Stay in place */
  z-index: 1; /* Sit on top */
  padding-top: 100px; /* Location of the box */
  left: 0;
  top: 0;
  width: 100%; /* Full width */
  height: 100%; /* Full height */
  overflow: auto; /* Enable scroll if needed */
  background-color: rgb(0,0,0); /* Fallback color */
  background-color: rgba(0,0,0,0.4); /* Black w/ opacity */
  padding-top: 300px;

}

/* Modal Content */
.modal-content {
  background-color: #fefefe;
  margin: auto;
  padding: 20px;
  border: 1px solid #888;
  width: 600px;
  border-radius: 10px;
  box-shadow: #393c76 -1px 2px 2px 1px;
}

#modalText {
  padding-top: 8px;
  padding-right: 5px;
  font-size: 18px;
  font-family: 'Poppins', sans-serif;
  color: rgb(24, 23, 23);
}

.modal-wrong {
border: 2px solid red;
}

.modal-correct {
  border: 2px solid green;
  }

/* The Close Button */
.close {
  color: #aaaaaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s;
}

.close:hover {
  color: #d41111;
}

.linkDownload {
    position: fixed;
    background-color: #d12322;
    margin: 20px;
    padding: 10px 10px;
    left: -0px;
    border-radius: 5px;
    bottom: -0px;
    font-size: 16px;
    font-weight: 400;
    color: #fff;
    text-decoration: none;
    text-transform: capitalize;
    transition: all 0.43s ease-in-out;
  }

.linkDownload i {
    padding-left: 7px;
  }

  .linkDownload:hover {
    text-decoration: none;
    background-color: black;
  }

@media (max-width: 700px){
    .box {
        width: 380px;
    }

    .input label {

        font-size: 18px;
    }

    .inputH label, .inputW label {
        font-size: 14px;
    }

    .input input, .inputH input, .inputW input{
        font-size: 24px;
    }

    .modal-content {
      width: 380px;
    }
}
```

`index.js`

```js
import { BMI_backend } from "../../declarations/BMI_backend";

var age = document.getElementById("age");
var height = document.getElementById("height");
var weight = document.getElementById("weight");
var male = document.getElementById("m");
var female = document.getElementById("f");
var form = document.getElementById("form");
let resultArea = document.querySelector(".comment");

var modalContent = document.querySelector(".modal-content");
var modalText = document.querySelector("#modalText");
var modal = document.getElementById("myModal");
var span = document.getElementsByClassName("close")[0];

document.getElementById("submit").addEventListener("click", async (e) => {
  e.preventDefault();
  const button = e.target;

  // const name = document.getElementById("name").value.toString();

  button.disabled = true;
  calculate();

  button.disabled = false;

  return false;
});

function calculate() {
  if (
    age.value == "" ||
    height.value == "" ||
    weight.value == "" ||
    (male.checked == false && female.checked == false)
  ) {
    modal.style.display = "block";
    modalText.innerHTML = `All fields are required!`;
  } else {
    countBmi();
  }
}

async function countBmi() {
  var p = [age.value, height.value, weight.value];
  if (male.checked) {
    p.push("male");
  } else if (female.checked) {
    p.push("female");
  }

  const bmi_data = await BMI_backend.calculateBMI(
    parseFloat(p[2]),
    parseFloat(p[1])
  );

  const bmi = bmi_data[0];
  const result = bmi_data[1];

  resultArea.style.display = "block";
  document.querySelector(
    ".comment"
  ).innerHTML = `You are <span id="comment">${result}</span>`;
  document.querySelector("#result").innerHTML = bmi.toFixed(2);
}

// When the user clicks on <span> (x), close the modal
span.onclick = function () {
  modal.style.display = "none";
};

// When the user clicks anywhere outside of the modal, close it
window.onclick = function (event) {
  if (event.target == modal) {
    modal.style.display = "none";
  }
};
```

`index.html`

```js
<!DOCTYPE html>
<html lang="en">
<!-- By Ali Aslan -->

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="main.css">
  <title>BMI Calculator</title>
</head>

<body>

  <div class="container">
    <div class="box">
      <h1>BMI Calculator</h1>
      <div class="content">


        <div class="input">
          <label for="height">Age</label>
          <input type="number" class="text-input" id="age" autocomplete="off" required min="1" max="100" />
        </div>

        <div class="gender">

          <label class="container">
            <input type="radio" name="radio" id="m">
            <p class="text">Male</p>
            <span class="checkmark"></span>
          </label>


          <label class="container">
            <input type="radio" name="radio" id="f">
            <p class="text">Female</p>
            <span class="checkmark"></span>
          </label>

        </div>

        <div class="containerHW">
          <div class="inputH">
            <label for="height">Height(cm)</label>
            <input type="number" id="height" required>
          </div>

          <div class="inputW">
            <label for="weight">Weight(kg)</label>
            <input type="number" id="weight" required>
          </div>
        </div>
        <button class="calculate" id="submit">Calculate BMI</button>
      </div>
      <div class="result">
        <p>Your BMI is</p>
        <div id="result">00.00</div>
        <p class="comment"></p>
      </div>
      <div class="footer">
      </div>
    </div>
  </div>
  <!-- The Modal -->
  <div id="myModal" class="modal">
    <!-- Modal content -->
    <div class="modal-content">
      <span class="close">&times;</span>
      <p id="modalText"></p>
    </div>
  </div>
</body>
</html>
```

## TIC-TAC-TOE GAME

`main.mo`

```js
actor {
  type BoxInputData = {
    box0 : Text;
    box1 : Text;
    box2 : Text;
  };

  public query func changeTurn(turn : Text) : async Text {
    var result : Text = "";
    if (turn == "X") {
      result := "0";
    } else {
      result := "X";
    };
    return result;
  };

  public func checkWin(data : BoxInputData) : async Bool {
    if ((data.box0 == data.box1) and (data.box2 == data.box1) and (data.box0 != "")) {
      return true;
    };
    return false;
  };
};
```

`main.css`

```js
@import url('https://fonts.googleapis.com/css2?family=Roboto:ital@1&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Baloo+Bhaina+2&display=swap');

*{
    margin: 0px;
    padding: 0px;
}

nav{
    background-color: black;
    color: white;
    height: 45px;
    font-size: 22px;
    display: flex;
    align-items: center;
    font-family: 'Roboto', sans-serif;
}

nav ul{
    list-style-type: none;
}

nav ul li{
    float: left;
    margin: 7px 20px;

}

.gameContainer{
    display: flex;
    justify-content: center;
    margin-top: 20px;
}

.container{
    display: grid;
    grid-template-columns: repeat(3,10vw);
    grid-template-rows: repeat(3,10vw);
    font-family: 'Roboto', sans-serif;
}

.box{
    border: 2px solid black;
    place-items: center;
    font-size: 8vw;
    cursor: pointer;
    display: flex;
    justify-content: center;
}

.container div:hover{
    background-color: rgb(220, 196, 243);
}

.gameinfo{
    margin-left: 20px;
    font-family: 'Baloo Bhaina 2', cursive;
    font-size: 20px
}

.imgbox img{
    width: 0;
    transition: width .1s ease-in-out;
}

.br-0{
    border-right: 0;
}

.bl-0{
    border-left: 0;
}

.bb-0{
    border-bottom: 0;
}

.bt-0{
    border-top: 0;
}

.gameinfo span{
    color: rgb(240, 80, 6);
    border: 3px solid black;
    border-radius: 9px;
    padding: 5px 20px;
    margin-left: 40px;
}

.gameinfo button{
    padding: 13px 20px;
    border-radius: 9px;
    background-color: black;
    color: whitesmoke;
}

.gameinfo h1{
    margin: 20px 5px;
    padding: 50px ;
    font-size: 3vw;
}

@media screen and (max-width: 900px) {
    .gameContainer{
        flex-wrap: wrap;
    }
    .gameinfo h1{
        font-size: 1.5rem;
        margin-top: 0;
    }
    .container{
        grid-template-columns: repeat(3,20vw);
        grid-template-rows: repeat(3,20vw);
    }
}
```

`index.html`

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="/main.css">
    <title>Tic Tac Toe</title>
</head>
<body>
    <header>
        <nav id='navbar'>
            <ul>
                <li>TicTacToeGame.com</li>
            </ul>
        </nav>
    </header>

    <div class="gameContainer">
        <div id="container" class="container">
            <div class="box bt-0 bl-0"><span class="boxtext"></span></div>
            <div class="box bt-0"><span class="boxtext"></span></div>
            <div class="box bt-0 br-0"><span class="boxtext"></span></div>
            <div class="box bl-0"><span class="boxtext"></span></div>
            <div class="box"><span class="boxtext"></span></div>
            <div class="box br-0"><span class="boxtext"></span></div>
            <div class="box bb-0 bl-0"><span class="boxtext"></span></div>
            <div class="box bb-0"><span class="boxtext"></span></div>
            <div class="box bb-0 br-0"><span class="boxtext"></span></div>
        </div>

        <div class="gameinfo">
            <h1>Tic Tac Toe Game</h1>
            <div id="status"></div>
            <div>
                <span class="info">Turns of X</span>
                <button id="reset">Reset</button>
            </div>
            <div class="imgbox">
                <img src="excit.gif" alt="">
            </div>
        </div>
    </div>

</body>
</html>

```

`index.js`

```js
import { tic_tac_toe_game_backend } from "../../declarations/tic_tac_toe_game_backend";

let turn = "X";
let gameOver = false;
let gameWorkingNow = false;
let status = document.getElementById("status");

// Function to change the turn
const changeTurn = async () => {
  let nextTurn = await tic_tac_toe_game_backend.changeTurn(turn);
  return nextTurn;
};

// Function to check for a win
const checkWin = async () => {
  gameWorkingNow = true;
  status.innerText = "Processing";
  let boxtext = document.getElementsByClassName("boxtext");
  let wins = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < wins.length; i++) {
    let data = {
      box0: boxtext[wins[i][0]].innerText,
      box1: boxtext[wins[i][1]].innerText,
      box2: boxtext[wins[i][2]].innerText,
    };
    let result = await tic_tac_toe_game_backend.checkWin(data);
    if (result) {
      document.querySelector(".info").innerText =
        "Player " + boxtext[wins[i][0]].innerText + " Won";
      gameOver = true;
      document
        .querySelector(".imgbox")
        .getElementsByTagName("img")[0].style.width = "200px";
      status.innerText = "";
      gameWorkingNow = false;
    }
  }

  status.innerText = "";
  gameWorkingNow = false;
};

// Game Logic
let boxes = document.getElementsByClassName("box");
Array.from(boxes).forEach((element) => {
  let boxtext = element.querySelector(".boxtext");
  element.addEventListener("click", async () => {
    if (boxtext.innerText === "" && !gameOver && !gameWorkingNow) {
      boxtext.innerText = turn;
      turn = await changeTurn();
      await checkWin();
      if (!gameOver) {
        document.getElementsByClassName("info")[0].innerText =
          "Turn for : " + turn;
      }
    }
  });
});

// Reset function
reset.addEventListener("click", (e) => {
  let boxtext = document.querySelectorAll(".boxtext");
  Array.from(boxtext).forEach((element) => {
    element.innerText = "";
  });
  turn = "X";
  document.getElementsByClassName("info")[0].innerText = "Turn for : " + turn;
  gameOver = false;
  document.querySelector(".imgbox").getElementsByTagName("img")[0].style.width =
    "0";
});
```

## AGE-CALCULATOR

`main.mo`

```js
actor {
  type TimeData = {
    day: Nat32;
    month: Nat32;
    year: Nat32;
  };

  type TimeDataInput = {
    date: Nat32;
    month: Nat32;
    year: Nat32;
  };

  public func ageCalculate(currentData : TimeDataInput, birthData: TimeDataInput) : async TimeData {
    var years = currentData.year - birthData.year;
    var months : Nat32 = 0;
    var days : Nat32 = 0;

    if(currentData.month >= birthData.month){
      months := currentData.month - birthData.month;
    }else{
      months := birthData.month - currentData.month;
      years -= 1;
      months := 12 - months;
    };

    if(currentData.date >= birthData.date){
      days := currentData.date - birthData.date;
    }else{
      days := birthData.date - currentData.date;
      days := days + (30 - currentData.date);
      months -= 1;
    };

    var data = {
      day = days;
      month = months;
      year = years;
    };
    return data;
  };
};

```

`main.css`

```js
body {
    font-family: sans-serif;
    font-size: 1.1rem;
}

img {
    max-width: 50vw;
    max-height: 25vw;
    display: block;
    margin: auto;
}

form {
    display: flex;
    justify-content: center;
    gap: 0.5em;
    flex-flow: row wrap;
    max-width: 40vw;
    margin: auto;
    align-items: baseline;
}

button[type="submit"] {
    padding: 5px 20px;
    margin: 10px auto;
    float: right;
}

body {
    font-family: Arial, sans-serif;
    background-color: #f5f5f5;
    margin: 0;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100vh;
}

.calculator {
    box-sizing: border-box;
    background-color: #fff;
    border-radius: 10px;
    box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.2);
    padding: 30px;
    text-align: center;
    max-width: 400px;
    width: 100%;
}

h1 {
    color: #333;
}

.input-container {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    margin-top: 20px;
}

.input-container label {
    margin-bottom: 10px;
    font-weight: bold;
}

.input-container input {
    width: 50%;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    outline: none;
}

#calculate-button {
    background-color: #007bff;
    color: #fff;
    border: none;
    border-radius: 5px;
    padding: 10px 20px;
    cursor: pointer;
    font-weight: bold;
    margin-top: 20px;
}

#result {
    margin-top: 20px;
    word-wrap: break-word;
}
```

`index.html`

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width" />
    <title>Age Calculator</title>
    <base href="/" />
    <link rel="icon" href="favicon.ico" />
    <link type="text/css" rel="stylesheet" href="main.css" />
  </head>
  <body>
    <main>
      <div class="calculator">
        <h2>Age Calculator</h2>
        <div class="input-container">
            <label for="birthdate">Enter your birthdate:</label>
            <input type="date" id="birthdate">
        </div>
        <button id="calculate-button">Calculate Age</button>
        <div id="result"></div>
    </div>
    </main>
  </body>
</html>

```

`index.js`

```js
import { age_calculator_backend } from "../../declarations/age_calculator_backend";

const calculateButton = document.getElementById("calculate-button");
const result = document.getElementById("result");

calculateButton.addEventListener("click", async () => {
  try {
    result.textContent = "";
    const birthTime = new Date(document.getElementById("birthdate").value);
    let currentTime = new Date();

    let currentYear = currentTime.getFullYear();
    let birthYear = birthTime.getFullYear();
    let currentMonth = currentTime.getMonth();
    let birthMonth = birthTime.getMonth();
    let currentDate = currentTime.getDate();
    let birthDate = birthTime.getDate();

    let currentData = {
      date: currentDate,
      month: currentMonth,
      year: currentYear,
    };

    let birthData = {
      date: birthDate,
      month: birthMonth,
      year: birthYear,
    };

    let data = await age_calculator_backend.ageCalculate(
      currentData,
      birthData
    );

    result.textContent = `${data.year} years ${data.month} months ${data.day} days`;
  } catch (error) {
    result.textContent = error;
  }
});
```


