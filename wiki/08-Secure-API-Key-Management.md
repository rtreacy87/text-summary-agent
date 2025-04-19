# Secure API Key Management

When working with LLM APIs, proper management of API keys is crucial for security. Exposing API keys can lead to unauthorized usage, potential data breaches, and unexpected charges. This section covers best practices and tools for securely managing your API keys.

## The Risks of Insecure API Key Handling

Before diving into solutions, let's understand the risks:

1. **Financial Exposure**: Unauthorized use of your API keys can result in unexpected charges
2. **Data Privacy**: Compromised keys may allow access to sensitive data
3. **Code Repository Leaks**: Keys committed to public repositories are quickly discovered by bots
4. **Logging Exposure**: Keys printed to logs or console output can be captured
5. **Insider Threats**: Keys shared among team members without proper controls

## Best Practices for API Key Management

Regardless of which tool you choose, follow these fundamental best practices:

1. **Never Hardcode Keys**: Never include API keys directly in your code
2. **Avoid Environment Variables in Code**: Don't set environment variables directly in your code
3. **Rotate Keys Regularly**: Change your API keys periodically
4. **Use Least Privilege**: Use keys with the minimum necessary permissions
5. **Monitor Usage**: Regularly check API usage for unusual patterns
6. **Set Rate Limits**: Configure rate limits on your API keys when possible
7. **Use Different Keys**: Use different keys for development, testing, and production

## API Key Management Solutions

Let's explore several solutions for securely managing API keys, comparing their strengths and weaknesses.

### 1. Pass Password Manager

[Pass](https://www.passwordstore.org/) (the "standard Unix password manager") is a command-line password management tool that uses GPG encryption.

#### Setup and Usage

1. **Install Pass**:
   ```bash
   # macOS
   brew install pass
   
   # Ubuntu/Debian
   sudo apt-get install pass
   
   # Fedora
   sudo dnf install pass
   ```

2. **Initialize Pass**:
   ```bash
   # Generate a GPG key if you don't have one
   gpg --full-generate-key
   
   # Initialize pass with your GPG key ID
   pass init your-gpg-key-id
   ```

3. **Store API Keys**:
   ```bash
   # Store your API keys
   pass insert api/google-ai
   pass insert api/openai
   pass insert api/anthropic
   ```

4. **Retrieve API Keys in Python**:
   ```python
   import subprocess
   
   def get_api_key(key_name):
       """
       Securely retrieve an API key from the pass password manager.
       
       Args:
           key_name: The name of the key in pass (e.g., 'api/google-ai')
           
       Returns:
           The API key as a string
       """
       try:
           # Run the pass command to retrieve the key
           result = subprocess.run(
               ['pass', key_name],
               capture_output=True,
               text=True,
               check=True
           )
           
           # Return the key (strip to remove trailing newline)
           return result.stdout.strip()
       except subprocess.CalledProcessError as e:
           print(f"Error retrieving API key: {e}")
           return None
   
   # Usage
   google_api_key = get_api_key('api/google-ai')
   ```

#### Advantages of Pass

- **Strong Encryption**: Uses GPG for encryption
- **Command-Line Integration**: Easy to use in scripts
- **Version Control**: Can be used with Git for versioning
- **Open Source**: Free and open-source
- **Unix Philosophy**: Follows the Unix philosophy of doing one thing well

#### Limitations of Pass

- **Setup Complexity**: Requires GPG key setup
- **Limited GUI**: Primarily command-line focused
- **Platform Limitations**: Best on Unix-like systems
- **Team Sharing**: Not designed for team sharing out of the box

### 2. Environment Variables with .env Files

Using environment variables with `.env` files is a common approach for managing API keys.

#### Setup and Usage

1. **Create a .env File**:
   ```
   # .env
   GOOGLE_API_KEY=your-google-api-key
   OPENAI_API_KEY=your-openai-api-key
   ANTHROPIC_API_KEY=your-anthropic-api-key
   ```

2. **Add .env to .gitignore**:
   ```
   # .gitignore
   .env
   ```

3. **Load Environment Variables in Python**:
   ```python
   from dotenv import load_dotenv
   import os
   
   # Load environment variables from .env file
   load_dotenv()
   
   # Get API keys
   google_api_key = os.environ.get('GOOGLE_API_KEY')
   openai_api_key = os.environ.get('OPENAI_API_KEY')
   anthropic_api_key = os.environ.get('ANTHROPIC_API_KEY')
   ```

#### Advantages of .env Files

- **Simplicity**: Easy to set up and use
- **Wide Support**: Supported by many frameworks and tools
- **No External Dependencies**: Minimal dependencies required
- **Local Development**: Great for local development
- **IDE Integration**: Good support in most IDEs

#### Limitations of .env Files

- **File Security**: .env files are stored in plaintext
- **Accidental Commits**: Risk of accidentally committing .env files
- **No Rotation**: No built-in key rotation
- **No Access Control**: No granular access control
- **No Audit Trail**: No logging of access to keys

### 3. Python Keyring

[Keyring](https://pypi.org/project/keyring/) is a Python library that provides access to the system keyring service.

#### Setup and Usage

1. **Install Keyring**:
   ```bash
   pip install keyring
   ```

2. **Store API Keys**:
   ```python
   import keyring
   
   # Store API keys
   keyring.set_password('text_summarizer', 'google_api_key', 'your-google-api-key')
   keyring.set_password('text_summarizer', 'openai_api_key', 'your-openai-api-key')
   keyring.set_password('text_summarizer', 'anthropic_api_key', 'your-anthropic-api-key')
   ```

3. **Retrieve API Keys**:
   ```python
   import keyring
   
   # Get API keys
   google_api_key = keyring.get_password('text_summarizer', 'google_api_key')
   openai_api_key = keyring.get_password('text_summarizer', 'openai_api_key')
   anthropic_api_key = keyring.get_password('text_summarizer', 'anthropic_api_key')
   ```

#### Advantages of Keyring

- **System Integration**: Uses the system's secure storage
- **Cross-Platform**: Works on Windows, macOS, and Linux
- **Python Native**: Easy to use in Python applications
- **No Files**: No need to manage .env files
- **Secure Storage**: Keys are stored securely by the OS

#### Limitations of Keyring

- **Limited to Local Machine**: Keys are stored on the local machine
- **No Remote Access**: Not suitable for server deployments
- **Backend Variations**: Different backends on different platforms
- **Setup Required**: Initial setup required for each machine
- **No Team Sharing**: Not designed for team sharing

### 4. HashiCorp Vault

[HashiCorp Vault](https://www.vaultproject.io/) is a tool for securely accessing secrets, such as API keys, passwords, and certificates.

#### Setup and Usage

1. **Install Vault**:
   Follow the [installation instructions](https://developer.hashicorp.com/vault/downloads) for your platform.

2. **Start Vault Server**:
   ```bash
   vault server -dev
   ```

3. **Store API Keys**:
   ```bash
   export VAULT_ADDR='http://127.0.0.1:8200'
   export VAULT_TOKEN='your-vault-token'
   
   vault kv put secret/text_summarizer/api google_api_key=your-google-api-key openai_api_key=your-openai-api-key anthropic_api_key=your-anthropic-api-key
   ```

4. **Retrieve API Keys in Python**:
   ```python
   import hvac
   
   # Initialize the Vault client
   client = hvac.Client(url='http://127.0.0.1:8200', token='your-vault-token')
   
   # Get API keys
   secret = client.secrets.kv.v2.read_secret_version(path='text_summarizer/api')
   api_keys = secret['data']['data']
   
   google_api_key = api_keys['google_api_key']
   openai_api_key = api_keys['openai_api_key']
   anthropic_api_key = api_keys['anthropic_api_key']
   ```

#### Advantages of HashiCorp Vault

- **Enterprise-Grade**: Designed for enterprise use
- **Access Control**: Fine-grained access control
- **Dynamic Secrets**: Support for dynamic secrets
- **Audit Trail**: Comprehensive audit logging
- **Key Rotation**: Built-in key rotation
- **High Availability**: Support for high availability
- **Team Sharing**: Designed for team use

#### Limitations of HashiCorp Vault

- **Complexity**: More complex to set up and maintain
- **Resource Intensive**: Requires more resources
- **Learning Curve**: Steeper learning curve
- **Overkill for Small Projects**: May be overkill for small projects
- **Infrastructure Required**: Requires infrastructure to run

### 5. AWS Secrets Manager or Google Secret Manager

Cloud providers offer managed secrets management services.

#### Setup and Usage (AWS Secrets Manager)

1. **Store API Keys**:
   ```python
   import boto3
   
   # Create a Secrets Manager client
   client = boto3.client('secretsmanager')
   
   # Store API keys
   client.create_secret(
       Name='text_summarizer/api_keys',
       SecretString='{"google_api_key":"your-google-api-key","openai_api_key":"your-openai-api-key","anthropic_api_key":"your-anthropic-api-key"}'
   )
   ```

2. **Retrieve API Keys**:
   ```python
   import boto3
   import json
   
   # Create a Secrets Manager client
   client = boto3.client('secretsmanager')
   
   # Get API keys
   response = client.get_secret_value(SecretId='text_summarizer/api_keys')
   secret = json.loads(response['SecretString'])
   
   google_api_key = secret['google_api_key']
   openai_api_key = secret['openai_api_key']
   anthropic_api_key = secret['anthropic_api_key']
   ```

#### Advantages of Cloud Secret Managers

- **Managed Service**: No infrastructure to maintain
- **High Availability**: Built-in high availability
- **Integration**: Tight integration with cloud services
- **Access Control**: Fine-grained access control
- **Audit Trail**: Comprehensive audit logging
- **Key Rotation**: Built-in key rotation
- **Team Sharing**: Designed for team use

#### Limitations of Cloud Secret Managers

- **Cost**: Usage-based pricing
- **Vendor Lock-in**: Tied to specific cloud provider
- **Network Dependency**: Requires network access
- **Complexity**: More complex than local solutions
- **Setup Required**: Initial setup required

## Comparison of API Key Management Solutions

| Feature | Pass | .env Files | Python Keyring | HashiCorp Vault | Cloud Secret Managers |
|---------|------|------------|----------------|-----------------|------------------------|
| **Ease of Setup** | Moderate | Very Easy | Easy | Complex | Moderate |
| **Security Level** | High | Low | High | Very High | Very High |
| **Team Sharing** | Limited | Limited | No | Yes | Yes |
| **Access Control** | Limited | No | No | Yes | Yes |
| **Audit Trail** | No | No | No | Yes | Yes |
| **Key Rotation** | Manual | Manual | Manual | Automated | Automated |
| **Cost** | Free | Free | Free | Free/Paid | Usage-based |
| **Local Development** | Yes | Yes | Yes | Yes | Limited |
| **Server Deployment** | Limited | Yes | No | Yes | Yes |
| **Cross-Platform** | Unix-focused | Yes | Yes | Yes | Yes |

## Implementing Secure API Key Management in the Text Summarizer Agent

Let's implement a secure API key management system for our Text Summarizer Agent using Pass:

1. **Create a Key Management Module**:

   Create a new file `src/key_manager.py`:

   ```python
   """
   Secure API key management for the Text Summarizer Agent.
   """
   
   import subprocess
   import os
   from typing import Optional
   
   class KeyManager:
       """
       Manages API keys securely using the pass password manager.
       Falls back to environment variables if pass is not available.
       """
       
       @staticmethod
       def get_api_key(key_name: str, env_var_name: Optional[str] = None) -> Optional[str]:
           """
           Get an API key securely.
           
           Args:
               key_name: The name of the key in pass (e.g., 'api/google-ai')
               env_var_name: The name of the environment variable to fall back to
               
           Returns:
               The API key as a string, or None if not found
           """
           # First try to get the key from pass
           try:
               result = subprocess.run(
                   ['pass', key_name],
                   capture_output=True,
                   text=True,
                   check=True
               )
               return result.stdout.strip()
           except (subprocess.CalledProcessError, FileNotFoundError):
               # If pass fails or is not installed, fall back to environment variables
               if env_var_name:
                   return os.environ.get(env_var_name)
               return None
       
       @staticmethod
       def get_google_api_key() -> Optional[str]:
           """Get the Google API key."""
           return KeyManager.get_api_key('api/google-ai', 'GOOGLE_API_KEY')
       
       @staticmethod
       def get_openai_api_key() -> Optional[str]:
           """Get the OpenAI API key."""
           return KeyManager.get_api_key('api/openai', 'OPENAI_API_KEY')
       
       @staticmethod
       def get_anthropic_api_key() -> Optional[str]:
           """Get the Anthropic API key."""
           return KeyManager.get_api_key('api/anthropic', 'ANTHROPIC_API_KEY')
   ```

2. **Use the Key Manager in Your Code**:

   Update your `src/text_summarizer_agent.py` to use the key manager:

   ```python
   from key_manager import KeyManager
   import google.generativeai as genai
   import openai
   from anthropic import Anthropic
   
   # Configure API clients securely
   google_api_key = KeyManager.get_google_api_key()
   if google_api_key:
       genai.configure(api_key=google_api_key)
   
   openai_api_key = KeyManager.get_openai_api_key()
   if openai_api_key:
       openai_client = openai.OpenAI(api_key=openai_api_key)
   
   anthropic_api_key = KeyManager.get_anthropic_api_key()
   if anthropic_api_key:
       anthropic_client = Anthropic(api_key=anthropic_api_key)
   ```

## Preventing API Key Exposure in Logs and Console Output

Even with secure storage, API keys can be accidentally exposed in logs or console output. Here's how to prevent this:

1. **Configure Logging to Redact Sensitive Information**:

   Create a custom log filter:

   ```python
   import logging
   import re
   
   class SensitiveDataFilter(logging.Filter):
       """Filter that redacts sensitive information from log records."""
       
       def __init__(self, patterns=None):
           super().__init__()
           self.patterns = patterns or [
               # API key patterns
               r'(api[_-]?key|apikey)["\']?\s*[:=]\s*["\']?([a-zA-Z0-9_\-]{20,})["\']?',
               r'(auth[_-]?token|token)["\']?\s*[:=]\s*["\']?([a-zA-Z0-9_\-]{20,})["\']?',
               # Add more patterns as needed
           ]
       
       def filter(self, record):
           if isinstance(record.msg, str):
               for pattern in self.patterns:
                   record.msg = re.sub(pattern, r'\1: [REDACTED]', record.msg)
           return True

   # Configure logging
   logger = logging.getLogger()
   logger.addFilter(SensitiveDataFilter())
   ```

2. **Create a Safe Print Function**:

   ```python
   def safe_print(message):
       """
       Print a message with sensitive information redacted.
       
       Args:
           message: The message to print
       """
       # Patterns to redact
       patterns = [
           # API key patterns
           r'(api[_-]?key|apikey)["\']?\s*[:=]\s*["\']?([a-zA-Z0-9_\-]{20,})["\']?',
           r'(auth[_-]?token|token)["\']?\s*[:=]\s*["\']?([a-zA-Z0-9_\-]{20,})["\']?',
           # Add more patterns as needed
       ]
       
       # Redact sensitive information
       for pattern in patterns:
           message = re.sub(pattern, r'\1: [REDACTED]', message)
       
       # Print the redacted message
       print(message)
   ```

3. **Monkey Patch Built-in Functions**:

   For comprehensive protection, consider monkey patching built-in functions:

   ```python
   import builtins
   import re
   
   # Store the original print function
   original_print = builtins.print
   
   def safe_print(*args, **kwargs):
       """
       A safe version of print that redacts sensitive information.
       """
       # Patterns to redact
       patterns = [
           # API key patterns
           r'(api[_-]?key|apikey)["\']?\s*[:=]\s*["\']?([a-zA-Z0-9_\-]{20,})["\']?',
           r'(auth[_-]?token|token)["\']?\s*[:=]\s*["\']?([a-zA-Z0-9_\-]{20,})["\']?',
           # Add more patterns as needed
       ]
       
       # Process each argument
       safe_args = []
       for arg in args:
           if isinstance(arg, str):
               # Redact sensitive information
               for pattern in patterns:
                   arg = re.sub(pattern, r'\1: [REDACTED]', arg)
           safe_args.append(arg)
       
       # Call the original print function with safe arguments
       original_print(*safe_args, **kwargs)
   
   # Replace the built-in print function
   builtins.print = safe_print
   ```

## Best Practices for Team Environments

When working in a team environment, consider these additional best practices:

1. **Use a Centralized Secret Manager**:
   - HashiCorp Vault or cloud provider secret managers
   - Implement role-based access control

2. **Implement a Secrets Rotation Policy**:
   - Regularly rotate API keys
   - Automate rotation when possible

3. **Provide Clear Documentation**:
   - Document how to securely access and use API keys
   - Include instructions for new team members

4. **Use CI/CD Secrets Integration**:
   - Integrate with CI/CD pipelines securely
   - Use environment-specific secrets

5. **Conduct Security Audits**:
   - Regularly audit code for hardcoded secrets
   - Use automated tools to detect leaked secrets

## Conclusion

Secure API key management is a critical aspect of developing applications that use LLM APIs. By following the best practices and using the appropriate tools described in this guide, you can protect your API keys from exposure and unauthorized use.

For the Text Summarizer Agent, we recommend:

1. **For Individual Development**:
   - Use Pass or Python Keyring for local development
   - Fall back to .env files with proper .gitignore configuration

2. **For Team Development**:
   - Use HashiCorp Vault or cloud provider secret managers
   - Implement proper access controls and audit logging

3. **For Production Deployment**:
   - Use cloud provider secret managers
   - Implement automated key rotation

Remember, the most secure API key is one that's never exposed in the first place. Always be vigilant about how you handle and store your API keys.

Continue to [index](index.md) to explore other aspects of the Text Summarizer Agent.
