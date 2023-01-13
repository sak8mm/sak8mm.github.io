---
layout: post
title: Excel Clean Up
image: "/posts/excel_clean.png"
tags: [Python, Cleaning, Excel]
---

I routinely receive cumulative data sets that is pulled automatically from another entity where it is written to an excel file with the extension of "xls" and sent out to those who need it.

I use one particular sheet in this excel workbook that I use python's pandas package to tabulate and manipulate the data with. However, there are other cases where it is more of a pain to do this and it would be easier to delete all unnecessary rows and columns. This way I can just go to work from that dataset for another purpose.

Normally, it would not be a problem just to do this manually but I get these every month and I have collected them for five years. So that is 60 different excel spreadsheets and I am certainly not going to do that because it is way too time intensive. 

So how do I achieve this? Originally I thought I could just use the openpyxl package (helps manipulate excel files) to address this. I started by loading the excel workbook like so:

```python
wb = xl.load_workbook('test_file.xls')
```

However I received the following error from python:

![alt text](/img/posts/InvalidFileException.png "Python did not like this!")


With this, I had to change my approach and rethink how I was going to do this so I came up with the below.

---

I used the following python packages to accomplish this:

```python
import openpyxl as xl
import os # This package allows us to use the operating system.
import win32com.client as win32 # This package is basically vba for python. It allows us to interact and automate Windows applications with python.
import xlrd # xlrd is a library for reading data and formatting information from Excel files in the historical .xls format.
```
With the necessary packages loaded I created the following function to handle one file to convert from an xls extension to an xlsx extension.

```python
# Used for individual xls to xlsx excel file converting.
def xls_to_xlsx (path, file):
    """This function converts excel xls files to xlsx."""
    fname = os.path.join(path, file)
    excel = win32.gencache.EnsureDispatch('Excel.Application')
    wb = excel.Workbooks.Open(fname)
    wb.SaveAs(fname + 'x', FileFormat = 51)
    wb.Close()
    excel.Application.Quit()
    os.remove(fname)
    
# Need to establish the directory that the file is in. 
p = os.getcwd() # Used as example. You will need to ensure your own path to excel file is put correctly here.
f = 'test_file.xls' # Used as an example. You will need to ensure the file name used for this is correct.

# Run the function to convert the single file.
xls_to_xlsx(p, f)
```

The function above is called 'xls_to_xlsx' and it takes two arguments, path and file. The path is the location of the directory the file is in and file, which is the name of the file to convert.

The 'fname' variable in the function stores the file path so python knows where to look and what to look for. The 'excel' variable essentially turns on the excel application on the operating system to be used.

The variable 'wb' is used as an object to open the file. From there the SaveAs() feature is used to save the file with the extension of xlsx, then close the workbook, stop excel application form working, and lastly, remove the old file with the extension of xls I don't want anymore.

Before I ran the function the file looked like this:

![alt text](/img/posts/Before_xls_run.png "xls extension")

After I ran the file I see it has the extension I want:

![alt text](/img/posts/after_xls_run.png "xlsx extension")


Now that I have the file with the extension I want, I can now use the openpyxl package to start manipulating the data in the files. To do this I created the following function:

```python
def create_short_tab(path, file):
    """This function copies first tab of file and deletes unnecessary rows."""
    fname = os.path.join(path, file)
    wb = xl.load_workbook(fname)
    ws = wb.worksheets[0]
    ws_copy = wb.copy_worksheet(ws).title = f'{ws.title} short'
    ws_copy = wb.worksheets[-1]
    for row in reversed(range(2, ws_copy.max_row + 1)):
        values = []
        for cell in ws_copy[row]:
            values.append(cell.value)  
        if 'Per Diem - Member' not in values:
                ws_copy.delete_rows(row)
    wb.save(fname) 
    
# Need to establish the directory that the file is in. 
p = os.getcwd() # Used as example. You will need to ensure your own path to excel file is put correctly here.
f = 'test_file.xlsx' # Used as an example. You will need to ensure the file name used for this is correct.

# Run the function to convert the single file.
create_short_tab(p, f)
```

As you can see this function is a little like the one before with the 'fname' and 'wb' variables used. However, I am going to be working on an excel sheet so the 'ws' variable is used to create that object for it.

From there, I created a copy of the worksheet I want to manipulate with openpyxl. So I copied the sheet in excel I want and now this copy is what I am working off of which is 'ws_copy'. The original is going to stay the same.

The next part was slightly tricky because I needed to have the function search for a certain wording in a row and if it did not exist in that row, then delete that row. Since this is an object we created and we are using openpyxl, there is an index associated with this. So when we start searching from the top down, the indexes will change and rows will be missed or deleted because of index renumbering from previous deletes.

To get around this, I started from the bottom in my 'for loop' using the 'reversed' function you see in the code. This way when a row is deleted the index renumbering will not have an impact on our loop since I am working my way up. 

We also have this searching cell by cell in a row for the value we are looking for. With each row a 'values' list is created and stores each cell's value in that list variable. Once that row is done, the program will see if the phrase 'Per Diem - Memeber' is in the 'values' list. If that phrase is not in that list, that row will be deleted. If the phrase is in the list then it will keep the row and move on to the next row. Finally, the file is saved.

So before the function is ran the excel sheet looked like this (notice the excel tab at the bottom):

![alt text](/img/posts/before_short_run.png "Snippet of the sheet before")

And then after the fun (notice the excel tabs at the bottom):

![alt text](/img/posts/after_short_run.png "Snippet of the sheet after")

As you can see, we kept the original sheet and have a copy of the original keeping only the rows we want. So know this will be easier to use pandas for this set of data.

All this is set up to run one instance when I receive the data file. So for each file I get every month, I will run it through these two functions. 

But remember, I told you I have 60 files to go through and surely I am not going to go through each function 60 times for a total of 120 runs right. I'm definitely not doing that so I used a 'for loop' for the 60 files I already have to convert to xlsx and get rid of the rows. 

So for that I used the following code on the 60 files:

```python
# Cycles through each file folder for the year and converts all xls files to xlsx files.
path = os.getcwd() # Gets us in the right directory location.
files = os.listdir() # Gets us a list of folders/files in teh directory location.
fy_folders = [f for f in files if f[:2] == 'FY'] # gets all the FY folders the files in with their FY.

for folder in fy_folders:
    p = os.path.join(path, folder)
    os.chdir(p)
    files = os.listdir()
    for file in files:
        if file[-3:] == 'xls':
            fname = os.path.join(p, file)
            excel = win32.gencache.EnsureDispatch('Excel.Application')
            wb = excel.Workbooks.Open(fname)
            wb.SaveAs(fname+'x', FileFormat = 51)
            wb.Close()
            excel.Application.Quit()
            os.remove(fname)  
```

And then for adding a new excel sheet and deleting the row I don't want for the 60 files, the following code was used:


```python
# Cycles through each file folder for the year and converts all xls files to xlsx files.
path = os.getcwd() # Gets us in the right directory location.
files = os.listdir() # Gets us a list of folders/files in teh directory location.
fy_folders = [f for f in files if f[:2] == 'FY'] # gets all the FY folders the files in with their FY.

for folder in fy_folders:
    p = os.path.join(path, folder)
    os.chdir(p)
    files = os.listdir()
    for file in files:
        fname = os.path.join(p, file)
        wb = xl.load_workbook(fname)
        ws = wb.worksheets[1]
        ws_copy = wb.copy_worksheet(ws).title = f'{ws.title} short'
        ws_copy = wb.worksheets[-1]
        for row in reversed(range(2, ws_copy.max_row + 1)):
            values = []
            for cell in ws_copy[row]:
                values.append(cell.value)  
            if 'Per Diem - Member' not in values:
                    ws_copy.delete_rows(row)
        wb.save(fname)     
```

Obviously this process was built in chuncks and worked out from there, but now I have a program that can take the data I receive and manipulate it in a way that is easier for me to use for a good starting point. Plus it saved me a lot of time vice doing it manually!