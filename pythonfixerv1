# Fixed version with proper agents framework detection
# Requirements: pip install streamlit openai-agents

import sys
import os
import shutil
import subprocess
import asyncio
import streamlit as st

# More robust import detection
def test_agents_import():
    """Test agents import in isolation"""
    try:
        from agents import Agent, Runner
        return True, Agent, Runner
    except Exception as e:
        return False, None, None

def test_tools_import():
    """Test tools import separately"""
    try:
        from agents.tools import LocalShellTool
        return True, LocalShellTool
    except Exception as e:
        try:
            # Alternative import path
            from agents import LocalShellTool
            return True, LocalShellTool
        except Exception as e2:
            return False, None

# Perform imports
AGENTS_SUCCESS, Agent, Runner = test_agents_import()
TOOLS_SUCCESS, LocalShellTool = test_tools_import()

# Fallback imports if agents framework fails
if not AGENTS_SUCCESS:
    from openai import AsyncOpenAI

# Debug information
print(f"🔍 Debug Info:")
print(f"   Agents framework available: {AGENTS_SUCCESS}")
print(f"   Tools available: {TOOLS_SUCCESS}")
print(f"   Python version: {sys.version}")

# Function to create agent dynamically based on provided API key
def create_agent(api_key: str):
    os.environ['OPENAI_API_KEY'] = api_key
    
    if AGENTS_SUCCESS:
        tools = []
        if TOOLS_SUCCESS and LocalShellTool:
            try:
                shell_tool = LocalShellTool()
                tools.append(shell_tool)
                print("✅ LocalShellTool created successfully")
            except Exception as e:
                print(f"⚠️ LocalShellTool creation failed: {e}")
        
        try:
            agent = Agent(
                name="PythonFixer",
                instructions="You are a helpful agent that debugs Python scripts by analyzing logs and fixing errors. Provide only the corrected Python code without explanations.",
                tools=tools
            )
            print("✅ Agent created with agents framework")
            return agent
        except Exception as e:
            print(f"❌ Agent creation failed: {e}")
            return AsyncOpenAI(api_key=api_key)
    else:
        print("📱 Using direct OpenAI API")
        return AsyncOpenAI(api_key=api_key)

# Function to monitor log file for errors
def monitor_log_for_errors(log_path: str) -> bool:
    if not os.path.exists(log_path):
        return False
    with open(log_path, 'r', encoding='utf-8', errors='ignore') as f:
        content = f.read().lower()
        return "error" in content or "traceback" in content

# Function to backup and version the script
def backup_and_version_script(script_path: str) -> str:
    base, ext = os.path.splitext(script_path)
    i = 2
    while True:
        versioned_path = f"{base}_v{i}{ext}"
        if not os.path.exists(versioned_path):
            shutil.copy(script_path, versioned_path)
            return versioned_path
        i += 1

# Function to fix broken script
async def fix_broken_script(agent, script_path: str, error_log: str) -> str:
    with open(script_path, 'r') as f:
        original_code = f.read()

    prompt = f"""Fix this Python script based on the error log. Return ONLY the corrected Python code without any explanations or markdown formatting.

ORIGINAL SCRIPT:
{original_code}

ERROR LOG:
{error_log}"""

    try:
        # Check if this is an agents framework agent
        if AGENTS_SUCCESS and hasattr(agent, 'name'):
            print("🤖 Using agents framework for fixing...")
            try:
                result = await Runner.run(agent, input=prompt)
                fixed_code = result.final_output
            except Exception as e:
                print(f"❌ Agents framework failed: {e}")
                # Fallback to direct API
                client = AsyncOpenAI(api_key=os.environ.get('OPENAI_API_KEY'))
                response = await client.chat.completions.create(
                    model="gpt-4o",
                    messages=[
                        {"role": "system", "content": "You are an expert Python developer. Fix broken code and return only the corrected Python code without explanations or markdown."},
                        {"role": "user", "content": prompt}
                    ],
                    temperature=0.1
                )
                fixed_code = response.choices[0].message.content.strip()
        else:
            print("📱 Using direct OpenAI API for fixing...")
            # Direct OpenAI API
            response = await agent.chat.completions.create(
                model="gpt-4o",
                messages=[
                    {"role": "system", "content": "You are an expert Python developer. Fix broken code and return only the corrected Python code without explanations or markdown."},
                    {"role": "user", "content": prompt}
                ],
                temperature=0.1
            )
            fixed_code = response.choices[0].message.content.strip()

        # Clean up markdown if present
        if fixed_code.startswith("```python"):
            fixed_code = fixed_code[9:]
        elif fixed_code.startswith("```"):
            fixed_code = fixed_code[3:]
        if fixed_code.endswith("```"):
            fixed_code = fixed_code[:-3]

        new_script_path = backup_and_version_script(script_path)
        with open(new_script_path, 'w') as f:
            f.write(fixed_code.strip())

        return new_script_path

    except Exception as e:
        st.error(f"Error fixing script: {e}")
        return ""

# Function to run the script and capture output
def run_script(script_path: str, log_path: str) -> int:
    try:
        with open(log_path, 'w') as log_file:
            process = subprocess.Popen(
                [sys.executable, script_path], 
                stdout=log_file, 
                stderr=subprocess.STDOUT,
                text=True
            )
            try:
                process.wait(timeout=60)
            except subprocess.TimeoutExpired:
                process.kill()
                return -1
            return process.returncode
    except Exception as e:
        with open(log_path, 'w') as log_file:
            log_file.write(f"Error running script: {e}")
        return -1

# Streamlit UI
st.set_page_config(page_title="🐍 ScriptFixer Agent", layout="wide")
st.title("🐍💥 Python ScriptFixer Agent")
st.caption("Automatically detects, fixes, and tests broken Python scripts with AI assistance.")

# Show framework status with better detection
if AGENTS_SUCCESS:
    st.success("✅ OpenAI Agents framework loaded successfully")
    if TOOLS_SUCCESS:
        st.info("🔧 Shell tools available - Enhanced functionality enabled")
    else:
        st.info("📱 Basic agents functionality available")
else:
    st.warning("📱 Using direct OpenAI API - Full functionality available")

api_key = st.text_input("🔑 Enter your OpenAI API Key", type="password")

# Always show instructions (moved outside the if/else block)
with st.expander("📖 How to Use - Step by Step Instructions", expanded=False):
    st.markdown("""
    ### 🚀 Quick Start Guide
    
    **Prerequisites:**
    - Python 3.8+
    - OpenAI API key ([Get one here](https://platform.openai.com/api-keys))
    
    **Steps:**
    1. **Enter your OpenAI API key** in the field above
    2. **Upload a broken Python script** (.py file) using the file uploader
    3. **Watch the magic happen** as AI analyzes and fixes your code
    4. **Download the working version** of your script
    
    ### 🧪 Want to Test It? Try These Examples:
    
    Create any of these broken scripts to test the fixer:
    """)
    
    # Test case tabs
    tab1, tab2, tab3 = st.tabs(["Import Error", "Syntax Error", "Variable Error"])
    
    with tab1:
        st.markdown("**Import Error Test** - Save as `test_import_error.py`:")
        st.code("""import nonexistent_module
print("This won't work")""", language='python')
    
    with tab2:
        st.markdown("**Syntax Error Test** - Save as `test_syntax_error.py`:")
        st.code("""print("Hello World"  # Missing closing parenthesis
x = 5
print(f"Value: {x}")""", language='python')
    
    with tab3:
        st.markdown("**Variable Error Test** - Save as `test_variable_error.py`:")
        st.code("""x = 5
y = 10
print(f"Sum: {x + z}")  # z is not defined""", language='python')
    
    st.markdown("""
    ### 🛠️ How It Works Behind the Scenes:
    
    1. **Script Execution** 🏃‍♂️ - Runs your uploaded script and captures any errors
    2. **Error Analysis** 🔍 - AI analyzes the error log and original code  
    3. **Code Generation** 🤖 - GPT-4 generates a fixed version of your script
    4. **Validation** ✅ - Automatically tests the fixed script to ensure it works
    5. **Delivery** 📦 - Provides the working code for download
    
    ### ⚠️ Important Notes:
    - Requires an OpenAI API key (usage charges apply)
    - Original scripts are automatically backed up before modification
    - Works best with Python scripts under 1000 lines
    - All processing is done securely through OpenAI's API
    """)

if api_key:
    uploaded_file = st.file_uploader("📂 Upload your broken Python script", type="py")

    if uploaded_file:
        script_path = "temp_uploaded_script.py"
        with open(script_path, "wb") as f:
            f.write(uploaded_file.read())

        log_path = "agent_log.txt"

        try:
            agent = create_agent(api_key)
            if AGENTS_SUCCESS:
                st.success("🤖 AI Agent created with full framework support")
            else:
                st.success("🤖 AI Agent created with direct API")
        except Exception as e:
            st.error(f"Failed to create agent: {e}")
            st.stop()

        with st.spinner("🚀 Running script and monitoring for errors..."):
            run_result = run_script(script_path, log_path)
            
            # Read and display log
            log_lines = []
            if os.path.exists(log_path):
                with open(log_path, 'r', encoding='utf-8', errors='ignore') as f:
                    log_lines = f.readlines()

            st.subheader("📜 Execution Log")
            if log_lines:
                st.code("".join(log_lines[-30:]), language='bash')
            else:
                st.info("No output captured")

            if run_result == 0 and not monitor_log_for_errors(log_path):
                st.success("✅ Script ran successfully with no errors.")
            else:
                st.warning("❌ Script failed. Attempting automatic fix...")
                
                log_content = ""
                if os.path.exists(log_path):
                    with open(log_path, 'r', encoding='utf-8', errors='ignore') as f:
                        log_content = f.read()

                with st.spinner("🤖 AI is analyzing and rewriting your script..."):
                    fixed_path = asyncio.run(fix_broken_script(agent, script_path, log_content))

                if fixed_path and os.path.exists(fixed_path):
                    st.success(f"✨ Script fixed and saved as: `{fixed_path}`")
                    
                    # Show fixed code
                    with open(fixed_path, 'r') as f:
                        fixed_code = f.read()
                    
                    st.subheader("🔧 Fixed Code")
                    st.code(fixed_code, language='python')
                    
                    st.download_button(
                        "⬇️ Download Fixed Script", 
                        data=fixed_code, 
                        file_name=f"fixed_{uploaded_file.name}",
                        mime="text/plain"
                    )

                    st.info("🧪 Retesting fixed script...")
                    with st.spinner("Testing fixed script..."):
                        result = run_script(fixed_path, log_path)
                        
                        if os.path.exists(log_path):
                            with open(log_path, 'r', encoding='utf-8', errors='ignore') as f:
                                fixed_log = f.readlines()

                            st.subheader("📜 New Test Results")
                            st.code("".join(fixed_log[-30:]), language='bash')

                        if result == 0 and not monitor_log_for_errors(log_path):
                            st.balloons()
                            st.success("🎉 Fixed script is now 100% operational!")
                        else:
                            st.error("💣 Fix attempt failed. Manual debugging recommended.")
                else:
                    st.error("Failed to generate fixed script. Please try again.")

else:
    st.info("👆 Please enter your OpenAI API key to get started.")

# Always show features section (moved outside the if/else block)
st.markdown("### ✨ Key Features:")
col1, col2 = st.columns(2)

with col1:
    st.markdown("- 🔍 **Automatic Error Detection**")
    st.markdown("- 🤖 **AI-Powered Code Fixing**") 
    st.markdown("- 🧪 **Automatic Testing**")

with col2:
    st.markdown("- 📁 **Version Backup System**")
    st.markdown("- ⬇️ **Easy Download**")
    st.markdown("- 🛡️ **Robust Fallbacks**")

# Cleanup
try:
    if os.path.exists("temp_uploaded_script.py"):
        os.remove("temp_uploaded_script.py")
except:
    pass
