# Uploading files to Azure Data Lake Storage Gen2

### **Summary:**  

This document showcases how to successfully upload files to an Azure Data Lake Storage Gen2 account using Python. I will be using a Shared Access Signature (SAS) token to assist me with uploading the files to my ADLS account. 

For my demo, I am using a Python app that first downloads two mp3 file sets of the famous Egyptian Qari **"Mahmoud Khalil Al-Hussary"**. Following that I will proceed with uploading the mp3 file sets to my ADLS account. Let's begin!

## Part1: Acquiring SAS Token from ADLS account
Within my ADLS Gen2 account I proceed with creating a container called "mp3". To acquire the SAS Token, you can do either the following:

- Right click the container name and click "Generate SAS"
- Or double click the container and under "Settings" click the the "Shared access tokens"

From there select the signing method, in my case I will be using the "Account key" and proceed with setting the permissions on the SAS token. For the purpose of uploading files to the container within the ADLS account, "Write" permission will be sufficient.  
Below is a snapshot of how it looks:


When you specify a datetime range on how long you want the permission to be active associated with the SAS token, proceed with clicking the blue button "Generate SAS token and URL" and copy the "Blob SAS Token". That token will be placed in a config file 
for local testing or Azure Key Vault for production based scenarios. 


## Part2: Download MP3 Files
So for my demo, I am using Python to read URLs from a txt file. Following that I have several functions in placed to prep everything for downloading the mp3 files to the local machine the app is running on. Here is a snippet of the Python code:
```py
    folder_names = os.getenv("FOLDER_NAMES")
    folder_names = folder_names.split(',')

    for folder in folder_names: 
        folder_creation(folder)

    file_path = os.getenv("TXT_FILE_PATH")
    url_list = read_urls(file_path)

    condition1 = os.getenv("COND1")
    condition2 = os.getenv("COND2")

    proceed = input("Do you wish to proceed with downloading the mp3 files at this time? (y/n): ")
    if proceed.lower() == "y":
        staging_downloads(url_list, folder_names, condition1, condition2)
    else:
        print("Download cancelled.")

```

Following that these are the key functions being utilized to assist with the operations:
```py
def read_urls(file_path):
    try:
        with open(file_path, 'r') as file:
            urls = [line.strip() for line in file.readlines()]
        return urls
    except FileNotFoundError:
        print(f"File not found: {file_path}")
        return []
    except Exception as e:
        print(f"Error reading file: {e}")
        return []
    
def staging_downloads(urls, folders, condition1, condition2):

    start_range = int(input("Enter the start range: "))
    end_range = int(input("Enter the end range: "))


    for url in urls:

        for i in range(start_range, end_range + 1):
            file_number = str(i).zfill(3)
            mp3_url = f'{url}{file_number}.mp3'
            
            if condition1 in url: 
                save_path = f'{folders[0]}\{file_number}.mp3'
                download_mp3(mp3_url, save_path)

            if condition2 in url:
                save_path = f'{folders[1]}\{file_number}.mp3'
                download_mp3(mp3_url, save_path)

def download_mp3(url, save_path):
    response = requests.get(url)

    if response.status_code == 200:
        with open(save_path, 'wb') as file:
            file.write(response.content)
        print(f"MP3 downloaded successfully to {save_path}")
    else:
        print(f"Failed to download MP3. Status code: {response.status_code}")
```
