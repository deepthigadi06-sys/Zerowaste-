const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");

const app = express();
app.use(cors());
app.use(express.json());

// MongoDB Connection
mongoose.connect("mongodb://127.0.0.1:27017/zerowaste")
.then(() => console.log("MongoDB Connected"))
.catch(err => console.log(err));

// Schema
const foodSchema = new mongoose.Schema({
  title: String,
  quantity: String,
  location: String,
  contact: String,
  createdAt: {
    type: Date,
    default: Date.now
  }
});

const Food = mongoose.model("Food", foodSchema);

// API Routes

// Get all food
app.get("/api/food", async (req, res) => {
  const food = await Food.find().sort({ createdAt: -1 });
  res.json(food);
});

// Add food
app.post("/api/food", async (req, res) => {
  const newFood = new Food(req.body);
  await newFood.save();
  res.json(newFood);
});

// Delete food
app.delete("/api/food/:id", async (req, res) => {
  await Food.findByIdAndDelete(req.params.id);
  res.json({ message: "Food removed" });
});

// Frontend (HTML + CSS + JS inside backend)

app.get("/", (req, res) => {
  res.send(`
  <!DOCTYPE html>
  <html>
  <head>
    <title>Zero Waste ♻</title>
    <style>
      body {
        font-family: Arial;
        background: #f4f4f4;
        text-align: center;
        padding: 20px;
      }
      h1 {
        color: green;
      }
      input {
        padding: 8px;
        margin: 5px;
      }
      button {
        padding: 8px 15px;
        background: green;
        color: white;
        border: none;
        cursor: pointer;
      }
      .card {
        background: white;
        padding: 15px;
        margin: 10px auto;
        width: 300px;
        border-radius: 8px;
        box-shadow: 0 2px 5px rgba(0,0,0,0.2);
      }
    </style>
  </head>
  <body>

    <h1>Zero Waste - Food Sharing ♻</h1>

    <div>
      <input id="title" placeholder="Food Name">
      <input id="quantity" placeholder="Quantity">
      <input id="location" placeholder="Location">
      <input id="contact" placeholder="Contact">
      <br>
      <button onclick="addFood()">Add Food</button>
    </div>

    <h2>Available Food</h2>
    <div id="foodList"></div>

    <script>
      const API = "/api/food";

      async function loadFood() {
        const res = await fetch(API);
        const data = await res.json();

        const list = document.getElementById("foodList");
        list.innerHTML = "";

        data.forEach(food => {
          list.innerHTML += \`
            <div class="card">
              <h3>\${food.title}</h3>
              <p><b>Quantity:</b> \${food.quantity}</p>
              <p><b>Location:</b> \${food.location}</p>
              <p><b>Contact:</b> \${food.contact}</p>
              <button onclick="deleteFood('\${food._id}')">Mark as Taken</button>
            </div>
          \`;
        });
      }

      async function addFood() {
        const title = document.getElementById("title").value;
        const quantity = document.getElementById("quantity").value;
        const location = document.getElementById("location").value;
        const contact = document.getElementById("contact").value;

        await fetch(API, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ title, quantity, location, contact })
        });

        document.getElementById("title").value = "";
        document.getElementById("quantity").value = "";
        document.getElementById("location").value = "";
        document.getElementById("contact").value = "";

        loadFood();
      }

      async function deleteFood(id) {
        await fetch(API + "/" + id, {
          method: "DELETE"
        });
        loadFood();
      }

      loadFood();
    </script>

  </body>
  </html>
  `);
});

// Start server
app.listen(5000, () => {
  console.log("Server running at http://localhost:5000");
});
