# Quick Start Guide - Python UCP Server

A simplified guide to get the UCP Python REST server running on your machine.

## TL;DR - Quick Commands (If Already Set Up)

**If you've already done the setup, just run:**

```bash
cd C:\Users\user\Documents\Workspace\ai\samples\rest\python\server
python -m uv run server.py --products_db_path=../ucp_test/products.db --transactions_db_path=../ucp_test/transactions.db --port=8182
```

Then visit: http://localhost:8182/.well-known/ucp

**First time? Follow the full setup below.**

---

## First Time Setup (Copy & Paste These Commands)

**Everything you already have is set up! Just run the server:**

```bash
# 1. Navigate to the server directory
cd C:\Users\user\Documents\Workspace\ai\samples\rest\python\server

# 2. Start the server (Option 1: Using venv directly - recommended)
.venv\Scripts\python.exe server.py --products_db_path=../ucp_test/products.db --transactions_db_path=../ucp_test/transactions.db --port=8182

# OR Option 2: Activate venv first, then run
.venv\Scripts\activate
python server.py --products_db_path=../ucp_test/products.db --transactions_db_path=../ucp_test/transactions.db --port=8182
```

**That's it!** The server should now be running on http://localhost:8182

To stop the server: Press `Ctrl+C`

---

## Setting Up From Scratch (For New Machines)

If you're setting this up on a fresh machine or the setup above didn't work, follow these detailed steps:

### Prerequisites

- **Python 3.10 or higher** (Python 3.11+ recommended)
- **Git** (to clone the SDK repository)
- **Internet connection** (for installing dependencies)

## Step-by-Step Setup

### 1. Install UV Package Manager

UV is a fast Python package manager that automatically manages virtual environments for you. Install it using pip:

**Windows:**
```bash
python -m pip install uv
```

**macOS/Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

> **Note on Virtual Environments:** UV automatically creates and manages `.venv` directories for each project when you run `uv sync`. You don't need to manually create or activate virtual environments - UV handles this for you!

### 2. Clone the UCP Python SDK

The server requires the UCP Python SDK. Clone it to the correct location:

```bash
# Navigate to the parent directory of the samples repository
cd ..

# Create sdk directory and clone the Python SDK
mkdir sdk
git clone https://github.com/Universal-Commerce-Protocol/python-sdk.git sdk/python

# Install SDK dependencies
cd sdk/python
python -m uv sync

# Return to samples directory
cd ../../samples/rest/python
```

**Directory Structure After This Step:**
```
your-workspace/
├── sdk/
│   └── python/          # UCP Python SDK
└── samples/
    └── rest/
        └── python/      # You are here
```

### 3. Install Server Dependencies

Navigate to the server directory and install dependencies:

```bash
cd server
python -m uv sync
```

This command:
1. Creates a virtual environment at `server/.venv` (if it doesn't exist)
2. Installs all dependencies into that virtual environment:
   - FastAPI (web framework)
   - Uvicorn (ASGI server)
   - SQLAlchemy (database ORM)
   - UCP SDK (commerce protocol types)
   - And other dependencies

> **Note:** When you run `python -m uv run <command>`, UV automatically uses the virtual environment it created. No need to activate it manually!

### 4. Initialize Test Databases

Create databases and populate them with flower shop test data:

**Windows:**
```bash
# Create database directory
mkdir ..\ucp_test

# Import test data
python -m uv run import_csv.py --products_db_path=../ucp_test/products.db --transactions_db_path=../ucp_test/transactions.db --data_dir=../test_data/flower_shop
```

**macOS/Linux:**
```bash
# Create database directory
mkdir -p /tmp/ucp_test

# Import test data
python -m uv run import_csv.py --products_db_path=/tmp/ucp_test/products.db --transactions_db_path=/tmp/ucp_test/transactions.db --data_dir=../test_data/flower_shop
```

This creates two SQLite databases with:
- Products (roses, tulips, etc.)
- Inventory levels
- Customer data
- Discount codes
- Shipping rates

### 5. Start the Server

**Windows:**
```bash
python -m uv run server.py --products_db_path=../ucp_test/products.db --transactions_db_path=../ucp_test/transactions.db --port=8182
```

**macOS/Linux:**
```bash
python -m uv run server.py --products_db_path=/tmp/ucp_test/products.db --transactions_db_path=/tmp/ucp_test/transactions.db --port=8182
```

You should see:
```
INFO:     Started server process [XXXXX]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8182 (Press CTRL+C to quit)
```

### 6. Verify the Server is Running

Open a new terminal and test the discovery endpoint:

```bash
curl http://localhost:8182/.well-known/ucp
```

You should receive a JSON response with the merchant's UCP profile, including capabilities and payment handlers.

## Testing the Server

### Available Test Data

The server is pre-loaded with flower shop test data:

**Products:**
- `bouquet_roses` - Bouquet of Red Roses ($35.00)
- `bouquet_tulips` - Spring Tulips ($30.00)
- `bouquet_sunflowers` - Sunflower Bundle ($25.00)
- `orchid_white` - White Orchid ($45.00)
- `gardenias` - Gardenias ($20.00)
- `pot_ceramic` - Ceramic Pot ($15.00)

**Discount Codes:**
- `10OFF` - 10% off your order
- `WELCOME20` - 20% off your order
- `FIXED500` - $5.00 off your order

### Option 1: Run the Test Client (Recommended)

The repository includes a Python test client that demonstrates a complete checkout flow:

**First, set up the client:**
```bash
# Open a new terminal (keep the server running)
cd C:\Users\user\Documents\Workspace\ai\samples\rest\python\client\flower_shop
python -m uv sync
```

**Then run it:**
```bash
# Using venv directly (recommended)
.venv\Scripts\python.exe simple_happy_path_client.py --server_url=http://localhost:8182

# OR activate venv first
.venv\Scripts\activate
python simple_happy_path_client.py --server_url=http://localhost:8182
```

**What it does:**
1. ✅ Discover merchant capabilities (calls `/.well-known/ucp`)
2. ✅ Create a checkout session with roses
3. ✅ Add a ceramic pot to the cart
4. ✅ Apply "10OFF" discount code (10% off)
5. ✅ Select shipping address and fulfillment option
6. ✅ Calculate final price ($58.50 after discount)

You'll see real-time logs showing each API call and response!

### Option 2: Manual API Testing

**Create a Checkout Session:**

```bash
curl -X POST http://localhost:8182/checkout-sessions \
  -H "UCP-Agent: profile=\"https://agent.example/profile\"" \
  -H "request-signature: test" \
  -H "idempotency-key: $(uuidgen)" \
  -H "request-id: $(uuidgen)" \
  -H "Content-Type: application/json" \
  -d '{
    "line_items": [
      {
        "item": {
          "id": "bouquet_roses",
          "title": "Red Rose"
        },
        "quantity": 2
      }
    ],
    "buyer": {
      "full_name": "Jane Doe",
      "email": "jane.doe@example.com"
    },
    "currency": "USD",
    "payment": {
      "instruments": [],
      "handlers": []
    }
  }'
```

**Apply a Discount Code:**

Replace `CHECKOUT_ID` with the ID from the previous response:

```bash
curl -X PUT http://localhost:8182/checkout-sessions/CHECKOUT_ID \
  -H "UCP-Agent: profile=\"https://agent.example/profile\"" \
  -H "request-signature: test" \
  -H "idempotency-key: $(uuidgen)" \
  -H "request-id: $(uuidgen)" \
  -H "Content-Type: application/json" \
  -d '{
    "discounts": {
      "codes": ["10OFF"]
    }
  }'
```

Available discount codes (see "Available Test Data" section above for full list):
- `10OFF` - 10% off
- `WELCOME20` - 20% off
- `FIXED500` - $5.00 off

## Available Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/.well-known/ucp` | GET | Discover merchant capabilities |
| `/checkout-sessions` | POST | Create a new checkout session |
| `/checkout-sessions/{id}` | GET | Retrieve checkout session |
| `/checkout-sessions/{id}` | PUT | Update checkout session |
| `/checkout-sessions/{id}/complete` | POST | Complete checkout and create order |
| `/orders/{id}` | GET | Retrieve order details |
| `/testing/simulate-shipping/{id}` | POST | Simulate order shipped event |

## Server Configuration

**Command-line Options:**

```bash
python -m uv run server.py --help
```

Key flags:
- `--port` - Server port (default: 8182)
- `--products_db_path` - Path to products database
- `--transactions_db_path` - Path to transactions database
- `--simulation_secret` - Secret for testing endpoints

## Stopping the Server

Press `Ctrl+C` in the terminal where the server is running.

## Virtual Environment Management

UV automatically creates and manages virtual environments:

- **Location:** Each project gets a `.venv` directory (e.g., `server/.venv`, `sdk/python/.venv`)
- **Activation:** Not required! Use `python -m uv run <command>` and UV handles it
- **Manual activation:** If you prefer traditional workflow:
  - Windows: `server\.venv\Scripts\activate`
  - macOS/Linux: `source server/.venv/bin/activate`
- **Check installed packages:** `python -m uv pip list` (from the project directory)

## Troubleshooting

### "Distribution not found" error

**Problem:** UV can't find the UCP SDK.

**Solution:** Ensure the SDK is cloned to the correct relative path (`../../../../sdk/python/` from the server directory).

### "Table already exists" error

**Problem:** Database already initialized.

**Solution:** Either delete the database files and re-run `import_csv.py`, or just start the server with the existing databases.

### Port already in use

**Problem:** Port 8182 is occupied.

**Solution:** Use a different port:
```bash
python -m uv run server.py --products_db_path=../ucp_test/products.db --transactions_db_path=../ucp_test/transactions.db --port=8183
```

### Python version error

**Problem:** Python version is too old.

**Solution:** Install Python 3.10 or higher. Check your version:
```bash
python --version
```

### Virtual environment issues

**Problem:** Commands not finding packages or Python executables.

**Solution:**
- Always use `python -m uv run <command>` to automatically use the correct virtual environment
- Or manually activate the virtual environment first (see "Virtual Environment Management" section above)

## What's Next?

- **Explore the API:** Try different combinations of products, discounts, and fulfillment options
- **Read the Full README:** See `server/README.md` for detailed documentation
- **Customize Products:** Modify `test_data/flower_shop/products.csv` and re-import
- **Understand UCP:** Visit https://ucp.dev for protocol specifications
- **Try the A2A Agent:** Check out `../../a2a/` for the AI-powered shopping agent

## Project Structure

```
rest/python/
├── QUICKSTART.md          # This guide
├── server/
│   ├── server.py          # FastAPI application entry point
│   ├── import_csv.py      # Database initialization script
│   ├── pyproject.toml     # Dependencies and configuration
│   ├── routes/            # API endpoint handlers
│   ├── services/          # Business logic
│   └── README.md          # Detailed documentation
├── client/
│   └── flower_shop/
│       └── simple_happy_path_client.py  # Test client
├── test_data/
│   └── flower_shop/       # CSV test data
└── ucp_test/              # SQLite databases (created during setup)
```

## Common Commands Reference

**Start server (from server directory):**
```bash
cd C:\Users\user\Documents\Workspace\ai\samples\rest\python\server

# Option 1: Using venv directly (recommended)
.venv\Scripts\python.exe server.py --products_db_path=../ucp_test/products.db --transactions_db_path=../ucp_test/transactions.db --port=8182

# Option 2: Using UV
python -m uv run server.py --products_db_path=../ucp_test/products.db --transactions_db_path=../ucp_test/transactions.db --port=8182
```

**Run test client:**
```bash
cd C:\Users\user\Documents\Workspace\ai\samples\rest\python\client\flower_shop

# First time only: install dependencies
python -m uv sync

# Then run
.venv\Scripts\python.exe simple_happy_path_client.py --server_url=http://localhost:8182
```

**Test discovery endpoint:**
```bash
curl http://localhost:8182/.well-known/ucp
```

**View server logs:**
Check the terminal where the server is running - you'll see all API requests in real-time!

**Reinitialize databases (if needed):**
```bash
cd C:\Users\user\Documents\Workspace\ai\samples\rest\python\server
python -m uv run import_csv.py --products_db_path=../ucp_test/products.db --transactions_db_path=../ucp_test/transactions.db --data_dir=../test_data/flower_shop
```

## License

Apache License 2.0 - Copyright 2026 UCP Authors
