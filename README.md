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
  2. After all is done, call:
    folderCheck("1st swimlane supplement")
