Code & Comments:



import os
import pandas as pd

columns = [
    "Council",
    "Data acquisition date",
    "Licence ID",
    "Licence Type",
    "Property Address",
    "Licensee Name",
    "Licensee Address",
    "Manager/Agent Name",
    "Manager/Agent Address",
    "People in the licence",
    "Households in the licence",
    "Storeys",
    "Bedrooms",
    "Status",
    "Accessibility on website",
    "Council website",
    "Council email",
    "Google drive link",
    "Date Applied",
    "Date Start",
    "Date End",
    "More fields"
]

string_columns = [
    'Council',
    'Licence ID',
    "Licence Type",
    "Property Address",
    "Licensee Name",
    "Licensee Address",
    "Manager/Agent Name",
    "Manager/Agent Address",
    "Status",
    "Accessibility on website",
    "Council website",
    "Council email",
    "Google drive link",
    "More fields"
]

integer_columns = [
    'People in the licence',
    'Households in the licence',
    'Storeys',
    'Bedrooms'
]

date_columns = [
    "Data acquisition date",
    "Date Applied",
    "Date Start",
    "Date End"
]

# check1: check if columns names have any problem
def overallCheck(df, folder_name):
    if list(df.columns) != columns:
        print(f"Column error in {folder_name}")
        return df, False
    return df, True


def colCheck(df, folder_name):
    # check1: remove spaces and line change symbols in columns
    df.columns = df.columns.str.strip()
    for column in string_columns:
        df[column] = df[column].replace('\n', ' ', regex=True)
        df[column] = df[column].replace('\r', ' ', regex=True)

    # check2: check if date columns are formatted correctly
    for date_col in date_columns:
        wrong_format_counts = 0
        for entry in df[date_col]:
            if not (pd.isna(entry) or entry == ""):
                try:
                    pd.to_datetime(entry, format="%Y-%m-%d")
                except (ValueError, TypeError):
                    wrong_format_counts += 1
        print(f"In column {date_col} there are {wrong_format_counts} wrongly formatted date entries in folder {folder_name}.")

    # check3: check if integer columns are formatted correctly
    for int_col in integer_columns:
        # Here I convert the column type to string and store integers as strings for stability.
        # This prevents integers from being unintentionally converted to floats when opened in Excel or other applications.
        # This way they can still be inserted into SQL with an integer column type.
        df[int_col] = df[int_col].astype(str)

        wrong_format_counts = 0
        for i, entry in enumerate(df[int_col]):
            if entry == "" or pd.isna(entry):
                df.at[i, int_col] = str(0)
            else:
                try:
                    df.at[i, int_col] = str(
                        int(float(entry)))  # Ensuring conversion from float to int and then to string
                except ValueError:
                    wrong_format_counts += 1
        if wrong_format_counts > 0:
            print(f"In column {int_col} there are {wrong_format_counts} non-integer entries in folder {folder_name}")

    # check4: check if string columns entry are valid
    valid_more_fields = df['More fields'].isin(['YES', 'NO'])
    if not valid_more_fields.all():
        print(f"Invalid entries in 'More fields' column in folder {folder_name}")

    valid_accessibility = df['Accessibility on website'].isin(['YES', 'NO'])
    if not valid_accessibility.all():
        print(f"Invalid entries in 'Accessibility on website' column in folder '{folder_name}'")

    valid_licence_type = df['Licence Type'].isin(['Mandatory', 'Additional', 'Selective', 'Mandatory/Additional'])
    if not valid_licence_type.all():
        print(f"Invalid entries in 'Licence Type' column in folder '{folder_name}'")

    return df

# This code makes sure the naming for source file and standardized file are standard too
# it also runs the previous codes on the standardized file
def folderCheck(folder_path):
    for councilFolder in os.listdir(folder_path):
        councilFolderPath = os.path.join(folder_path, councilFolder)
        if not os.path.isdir(councilFolderPath):
            continue
        print(f"Now checking {councilFolder}: ")
        for dataFolder in os.listdir(councilFolderPath):
            dataFolderPath = os.path.join(councilFolderPath, dataFolder)
            if dataFolder == 'source':
                num_source = 0
                for filename in os.listdir(dataFolderPath):
                    num_source += 1
                    if len(os.listdir(dataFolderPath)) == 0:
                        print(f"There is no file in source folder in {councilFolder}")
                    if "." in filename:
                        file_name, ext = os.path.splitext(filename)
                        newname = f"{councilFolder}_{num_source}.{ext[1:]}"
                        os.rename(os.path.join(dataFolderPath, filename), os.path.join(dataFolderPath, newname))
                    else:
                        print(f"Invalid file name in source folder of {councilFolder}: {filename}")

            if dataFolder == 'standardized':
                csv_files = [file for file in os.listdir(dataFolderPath) if file.endswith('.csv')]
                if not csv_files:
                    print(f"There is no file in standardized folder in {councilFolder}")
                    continue

                standardized_filename = f"{councilFolder}.csv"
                if len(csv_files) > 1:
                    dfs = [pd.read_csv(os.path.join(dataFolderPath, file)) for file in csv_files]
                    df = pd.concat(dfs, ignore_index=True)
                else:
                    df = pd.read_csv(os.path.join(dataFolderPath, csv_files[0]))

                df, valid = overallCheck(df, councilFolder)
                if valid:
                    df = colCheck(df, councilFolder)

                df.to_csv(os.path.join(dataFolderPath, standardized_filename), index=False)


folderCheck("1st swimlane supplement")



How to use it?

1.Make sure the standardized files are in the following structure:
  -- mainfolder
    -- E06000032 (these are individual files for each council)
      -- source
        -- sourcefile1
        -- sourcefile2
        ...
      -- standardized
        -- standardizedfile1.csv
        -- standardizedfile1.csv
        ...
    -- E09000004
    ...
  Caution: please name the individual council file by their sequence, and  the standardized file and source will be renamed accordingly
  2. Call:
    folderCheck("1st swimlane supplement")
