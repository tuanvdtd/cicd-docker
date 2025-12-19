import express from 'express';
import mongoose from 'mongoose';
import cors from 'cors';

const app = express();
app.use(express.json());
app.use(cors());

const PORT = 3001;
// user: 20225423
// Kết nối đến MongoDB Atlas
const MONGODB_URI = 'mongodb+srv://20225423:20225423@cluster0.7pg6nb9.mongodb.net/?appName=Cluster0';
const DB_NAME = 'it4409';

// Connect to MongoDB Atlas

mongoose.connect(MONGODB_URI, {
	dbName: DB_NAME,
})
	.then(() => {
    // Connect được thì mới chạy server
		console.log('Connected to MongoDB Atlas');
		app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
	})
	.catch((err) => {
		console.error('Failed to connect to MongoDB Atlas:', err);
		process.exit(1);
	});

app.get('/', (req, res) => {
	res.json({ status: 'ok' });
});

// Tạo model user
const userSchema = new mongoose.Schema({
	name: {
    type: String,
    minlength: [2, 'Tên phải có ít nhất 2 ký tự'],
    maxlength: 50,
    required: [true, 'Tên không được để trống'],
    trim: true,
  },
	age: {
    type: Number,
    required: [true, 'Tuổi không được để trống'],
    min: [0, 'Tuổi phải lớn hơn bằng 0'],
  },
	email: {
    type: String,
    required: [true, 'Email không được để trống'],
    match: [/^\S+@\S+\.\S+$/, 'Email không hợp lệ'],
  },
  address: {
    type: String,
    trim: true,
  }
},{
  timestamps: true,
  collection: 'Tuan.DT225423'
});

const User = mongoose.model('User', userSchema);

// route user
app.get('/users', async (req, res) => {
  try {
    // lấy page và limit
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;
    const skip = (page - 1) * limit;
    const search = req.query.search || '';

    // query tìm kiếm theo name hoặc email
    const query = [
      { name: { $regex: search, $options: 'i' } },
      { email: { $regex: search, $options: 'i' } },
      { address: { $regex: search, $options: 'i' } },
    ];
    
    let users;
    let totalCount;
    if (search) {
      users = await User.find({ $or: query }).lean();
      [users, totalCount] = await Promise.all([await (await User.find({ $or: query }).skip(skip).limit(limit)).sort({name: 1}).lean(), await User.countDocuments({ $or: query })]);
    }
    else {
      [users, totalCount] = await Promise.all([await User.find().skip(skip).limit(limit).lean(), await User.countDocuments()]);
    }
    return res.json({users, totalCount, page, limit});
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
} 
)

app.post('/users', async (req, res) => {
  try {
    const { name, age, email, address } = req.body;
    let newUser
    if (!name || !age || !email) {
      return res.status(400).json({ error: 'Name, age and email are required' });
    }
    const emailExists = await User.findOne({ email });
    if (emailExists) {
      return res.status(400).json({ error: `Email ${email} already exists` });
    }
    if (isNaN(age) || age < 0) {
      return res.status(400).json({ error: 'Age must be a non-negative number' });
    }

    if (address) {
      newUser = await User.create({ name, age, email, address });
    } else {
      newUser = await User.create({ name, age, email });
    }
    res.status(201).json(newUser);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.get('/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.status(200).json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.put('/users/:id', async (req, res) => {
  try {
    const { name, age, email, address } = req.body;
    const user = await User.findByIdAndUpdate(req.params.id, { name, age, email, address }, { new: true });
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.status(200).json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.delete('/users/:id', async (req, res) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id); 
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json({ message: 'User deleted successfully', data: user });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

export default app;

