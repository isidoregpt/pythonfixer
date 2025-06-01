# 🐍💥 Python ScriptFixer Agent

An intelligent Python script debugging tool that automatically detects, analyzes, and fixes broken Python scripts using OpenAI's GPT-4 and the OpenAI Agents framework.

## ✨ Features

- 🔍 **Automatic Error Detection** - Runs scripts and captures all error output
- 🤖 **AI-Powered Code Fixing** - Uses GPT-4 to analyze and fix Python errors
- 🧪 **Automatic Testing** - Tests fixed scripts to ensure they work
- 📁 **Version Control** - Automatically creates backups of original scripts
- ⬇️ **Easy Download** - Download fixed scripts instantly
- 🛡️ **Robust Fallbacks** - Works with both Agents framework and direct OpenAI API

## 🚀 Quick Start

### Prerequisites

- Python 3.8+
- OpenAI API key ([Get one here](https://platform.openai.com/api-keys))

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/python-scriptfixer-agent.git
   cd python-scriptfixer-agent
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Run the application**
   ```bash
   streamlit run fixed_streamlit_app.py
   ```

4. **Open your browser** and navigate to `http://localhost:8501`

### Usage

1. Enter your OpenAI API key
2. Upload a broken Python script (.py file)
3. Watch as the AI analyzes and fixes your code
4. Download the working version!

## 🛠️ How It Works

1. **Script Execution**: Runs your uploaded script and captures any errors
2. **Error Analysis**: AI analyzes the error log and original code
3. **Code Generation**: GPT-4 generates a fixed version of your script
4. **Validation**: Automatically tests the fixed script to ensure it works
5. **Delivery**: Provides the working code for download

## 📁 Project Structure

```
python-scriptfixer-agent/
├── fixed_streamlit_app.py    # Main application
├── requirements.txt          # Dependencies
├── README.md                # This file
└── .gitignore               # Git ignore file
```

## 🔧 Configuration

### Environment Variables

You can set your OpenAI API key as an environment variable:

```bash
export OPENAI_API_KEY="your-api-key-here"
```

Or create a `.env` file:
```
OPENAI_API_KEY=your-api-key-here
```

## 🧪 Example Test Cases

Create these files to test the script fixer:

**Import Error Test** (`test_import_error.py`):
```python
import nonexistent_module
print("This won't work")
```

**Syntax Error Test** (`test_syntax_error.py`):
```python
print("Hello World"  # Missing closing parenthesis
x = 5
print(f"Value: {x}")
```

**Variable Error Test** (`test_variable_error.py`):
```python
x = 5
y = 10
print(f"Sum: {x + z}")  # z is not defined
```

## 🚀 Deployment

### Streamlit Cloud

1. Push your code to GitHub
2. Connect your GitHub repo to [Streamlit Cloud](https://streamlit.io/cloud)
3. Add your `OPENAI_API_KEY` to the secrets management
4. Deploy!

### Local Development

```bash
# Install in development mode
pip install -e .

# Run with debug mode
streamlit run fixed_streamlit_app.py --logger.level debug
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## ⚠️ Important Notes

- Requires an OpenAI API key (usage charges apply)
- Uploaded scripts are processed locally and sent to OpenAI for analysis
- Original scripts are automatically backed up before modification
- Works best with Python scripts under 1000 lines

## 🆘 Troubleshooting

### Common Issues

**"Agents framework not available"**
- Try: `pip uninstall openai-agents && pip install openai-agents`
- The app will work fine with the direct API fallback

**"No module named 'streamlit'"**
- Run: `pip install -r requirements.txt`

**API Key Issues**
- Ensure your OpenAI API key is valid and has sufficient credits
- Check if the key is properly entered (no extra spaces)

## 🙏 Acknowledgments

- Built with [Streamlit](https://streamlit.io/)
- Powered by [OpenAI GPT-4](https://openai.com/)
- Uses [OpenAI Agents Framework](https://github.com/openai/openai-agents-python)

---

Made with ❤️ for Python developers who want AI-powered debugging assistance.
