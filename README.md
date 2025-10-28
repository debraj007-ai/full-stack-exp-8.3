import express from "express";
import mongoose from "mongoose";
import cors from "cors";

// ===== CONNECT TO MONGODB =====
mongoose.connect("mongodb://localhost:27017/ecommerceDB", {
  useNewUrlParser: true,
  useUnifiedTopology: true
})
.then(() => console.log(" MongoDB Connected"))
.catch(err => console.error(" MongoDB Error:", err));

// ===== SCHEMA (Nested Document Design) =====
const variantSchema = new mongoose.Schema({
  color: String,
  size: String,
  price: Number,
  stock: Number
});

const productSchema = new mongoose.Schema({
  name: { type: String, required: true },
  brand: String,
  description: String,
  variants: [variantSchema]
});

const categorySchema = new mongoose.Schema({
  name: { type: String, required: true },
  description: String,
  products: [productSchema]
});

const Category = mongoose.model("Category", categorySchema);

// ===== EXPRESS SETUP =====
const app = express();
app.use(express.json());
app.use(cors());

// ===== ROUTES =====

//  Add Category (with nested products & variants)
app.post("/categories", async (req, res) => {
  try {
    const category = new Category(req.body);
    res.status(201).json(await category.save());
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

//  Get All Categories (nested structure)
app.get("/categories", async (req, res) => {
  res.json(await Category.find());
});

//  Get Single Category
app.get("/categories/:id", async (req, res) => {
  try {
    const cat = await Category.findById(req.params.id);
    cat ? res.json(cat) : res.status(404).json({ message: "Not found" });
  } catch {
    res.status(400).json({ message: "Invalid ID" });
  }
});

//  Update Category (or nested product info)
app.put("/categories/:id", async (req, res) => {
  try {
    const updated = await Category.findByIdAndUpdate(req.params.id, req.body, { new: true });
    updated ? res.json(updated) : res.status(404).json({ message: "Not found" });
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

//  Delete Category
app.delete("/categories/:id", async (req, res) => {
  try {
    const deleted = await Category.findByIdAndDelete(req.params.id);
    deleted ? res.json({ message: "Deleted successfully" }) : res.status(404).json({ message: "Not found" });
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

// ===== START SERVER =====
const PORT = 5000;
app.listen(PORT, () => console.log(`Server running on http://localhost:${PORT}`));
