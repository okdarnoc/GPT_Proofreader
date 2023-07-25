# OpenAI Model Text Improvement Script

This script uses the OpenAI GPT-3.5 language model to improve the text content of files in a source directory and save the improved content to a target directory.

## Setup

1. Install the required dependencies:
   - `openai`
   - `json`

2. Set up your OpenAI API key:
   - Enter your OpenAI API key when prompted by the script.

## Usage

1. Provide the source and target directories and the file containing the list of processed files.
2. Set the temperature and model to be used.
3. Define the prompt message for text improvement.

```python
import os
import openai
import json
import time

# Set OpenAI API key
openai.api_key = input("Please enter your OpenAI API key: ")

# prompt for source and target directories and processed files list file location
source_dir = input('Please enter the path to the source directory: ')
target_dir = input('Please enter the path to the target directory: ')

processed_files_file = input('Please enter the path to the processed files list file (will be created if it does not exist): ')
if not processed_files_file:
    processed_files_file = os.path.join(target_dir, 'processed_files.json')

# Prompt for temperature, model, and prompt message
temperature = float(input('Please enter the temperature for the model (between 0 and 1): '))
model = input('Please enter the model to be used (gpt-4, etc.): ')
prompt_message = input('Please enter the prompt message: ')

# load the list of processed files and unprocessable files
try:
    with open(processed_files_file, 'r') as file:
        file_data = json.load(file)
    processed_files = file_data.get('processed_files', [])
    unprocessable_files = file_data.get('unprocessable_files', [])
except FileNotFoundError:
    processed_files = []
    unprocessable_files = []

# make sure the target directory exists
os.makedirs(target_dir, exist_ok=True)

def get_completion(prompt, model=model, temperature=temperature):
    messages = [
        {
            "role": "system", 
            "content": prompt_message
        },
        {
            "role": "user", 
            "content": prompt
        }
    ]

    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=temperature,
    )

    return response.choices[0].message["content"]

for filename in os.listdir(source_dir):
    # we only want to process .txt files and those that haven't been processed yet or are unprocessable
    if filename.endswith('.txt') and filename not in processed_files and filename not in unprocessable_files:
        try:
            # read the original file
            with open(os.path.join(source_dir, filename), 'r', encoding='utf-8') as file:
                content = file.read()
                
            # get the improved content
            improved_content = get_completion(content)
            
            # write the improved content to a new file in the target directory
            with open(os.path.join(target_dir, filename), 'w', encoding='utf-8') as file:
                file.write(improved_content)
                
            # add the file to the list of processed files
            processed_files.append(filename)
            print(f"File {filename} processed successfully.")

        except Exception as e:
            # if there's an error processing the file, add it to the list of unprocessable files
            unprocessable_files.append(filename)
            print(f"File {filename} could not be processed. Error: {str(e)}")

        finally:
            # save the updated list of processed and unprocessable files
            with open(processed_files_file, 'w') as file:
                json.dump({'processed_files': processed_files, 'unprocessable_files': unprocessable_files}, file)

            # sleep for 10 seconds
            time.sleep(12)
```

##Output
The script processes .txt files in the source directory.
Improved content is saved to the target directory.

##Additional Notes
The script uses the OpenAI Chat Completion API to improve the text content.
If there are any errors processing a file, it is added to the list of unprocessable files.

##Disclaimer
This script assumes you have the necessary rights to use the OpenAI GPT-3.5 API and to access the files in the provided directories.

Please refer to the OpenAI API documentation for more information about API usage and authentication.

Note: The script is set to sleep for 12 seconds after processing each file to avoid API rate-limiting issues.
