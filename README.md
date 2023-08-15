# NextJS13-Sigma-Todo-List

Structure
├── components/
│   ├── EntryForm.jsx
│   ├── Nav.jsx
│   ├── TodoForm.jsx
│   ├── TodoItem.jsx
│   └── TodoList.jsx
├── app/
│   ├── api/
│   │ 	├── auth
│   │	│ 	├── [...nextauth]
│   │	│ 	│  	   └── route.js
│   │	│ 	├── hello
│   │	│ 	│  	   └── route.js
│   │	│ 	├── register
│   │	│ 	│  	   └── route.js
│   │	│ 	├── session
│   │	│ 	│  	   └── route.js
│   │ 	├── todos/
│   │	│ 	├── [id]/
│   │	│ 	│  	   └── route.js
│   │	│ 	├── new/
│   │	│ 	│  	   └── route.js
│   │	│ 	└── route.js
│   ├── entry/
│   │ 	└── page.js
│   ├── todos/
│   │   └── page.js
│   ├── layout.js
│   └── page.js
├── db/
│   ├── models/
│   │ 	├── TodoList.js
│   │ 	└── User.js
│   └── db.js
├── public/
│   ├── favicon.ico
│   └── ...
├── redux/
│   ├── actions/
│   │   ├── authActions.js
│   │   ├── todoActions.js
│   │   └── ...
│   ├── reducers/
│   │   ├── authReducer.js
│   │   ├── todoReducer.js
│   │   └── ...
│   ├── store.js
│   └── ...
└── styles/
    ├── globals.css
    └── tailwind.css
Credential authentication with next-auth
Database
npm i mongoose next-auth next-connect validator bcrypt
Create a user model with mongoose
Create dbConnect()
Backend
Set up endpoints: api/auth/register.js and _api/auth/[...nextauth].js
Create (API route) handler with next-connect @/utils/handler.js
Use handler to create register.js for new user api/auth/register.js
@[...nextauth].js : create a NextAuth function includes:
session: Enable JSON Web Tokens
providers: use SSO or credentials(email and password)
callback: Add a customized accessToken and user info
pages: '/entry'
Frontend:
import {useSession} from "next-auth/react" on todosPage and homePage
Use useSession() hook to extract user.name, then pass to as a props.
Save accessToken to localStorage. //***Todo: Optimized later with refreshToken saved to cookies
GET/PATCH/POST/DELETT/UPDATE todos with token
On client-side, and add accessToken to headers: { 'Authorization': accessToken} GET/PATCH/POST/DELETT/UPDATE
On server-side:
create api/auth/middlewares/requireAtth.js
extract token from the header:
    const authorizationHeader =
    req.headers instanceof Headers
      ? req.headers.get('authorization')
      : req.headers.authorization
  let token = authorizationHeader?.split(' ')[1];
Decode the token
const decodedToken = verify(token, secretKey);
extract user._id and add to req
req.user = decodedToken.sub;
NOTE: cannot get the [id] from url. For now use this way
const url = new URL(req.url, `http://${req.headers.host}`);
const id = url.pathname.split('/').pop();
Create a simple modal to addNote
Add the "note" field to your MongoDB model:
import { Schema, model, models }  from 'mongoose';

const TodoSchema = new Schema({
 //...existing fields
 note: {
   type: String,
   default: ''
 },
 //...existing fields
});

const Todo = models.Todo || model('Todo', TodoSchema);
export default Todo;
For the PATCH request in the api/todos/[id]/route.js file, include the "note" field:
export const PATCH = requireAuth(async(req ) => {
 const { title, priority , completed, note } = await req.json() //added note here
    //...existing fields

 try{
 await dbConnect()
 const foundTodo = await Todo.findById({ _id: id, user: req.user })
 if(!foundTodo) return new Response(`Todo does not exist`, {status: 404})
    //...existing fields
 if(note) foundTodo.note = note  // update note field here

 await foundTodo.save()
 return new Response(JSON.stringify(foundTodo),{status: 200})
 }catch(err){

 }
})
Use the browser's built-in prompt function:
const handleNote = (id) => {
 let note = prompt("Please enter your note:"); // shows a simple input popup
 if (note != null) {
   onAddNote(id, note); // calls function to save note
 }
}
Add onAddNote function
const onAddNote = (id, note) => {
 fetch(`/api/todos/${id}`, {
   method: 'PATCH',
   headers: {
     'Content-Type': 'application/json',
   },
   body: JSON.stringify({ note }),
 })
 .then(response => response.json())
 .then(data => {
   // do something with the data
 })
 .catch((error) => {
   console.error('Error:', error);
 });
}
Create a sticker note
Database
Create a demo database on MongoAtlas
Connect MongoAtlas with MongoCompass
Use MongoCompass to add Demo data
