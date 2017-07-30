#### Print variables in scope with some filter
```python
print([v for v in globals().keys() if (v.startswith('df') or v.startswith('var') or v.startswith('f'))])
```

#### List files in current directory or a subdirectory of current directory
```python
def listFilesInCurrent(dir=""):
    from subprocess import check_output
    platformText = (platform.platform(terse=1)).lower()
    if('windows' in platformText):
        print(check_output("cmd /k dir " + dir + " /b").decode(sys.stdout.encoding))
    elif('linux' in platformText):
        print(check_output(["ls", ['' if dir == "" else ("../" + dir)]]).decode("utf8"))
```

#### Print Markdown in IPython Jupyter Notebook
```python
def printmd(string):
    from IPython.display import Markdown, display
    display(Markdown(string))
```

#### Shrink pandas dataframe
```python
# Doesn't shrink columns with NANs, Shrinks inplace
def shrinkDf(df, verbose, dispInfo):
    initialSize = df.memory_usage().sum()
    for col in df.columns:
        if (df[col].dtype != object) and (np.isfinite(df[col]).all()):
            if(verbose==True):
                print("Column: ", col, ", original dtype: ", df[col].dtype)
                
            IsInt = False
            maxValue = df[col].max()
            minValue = df[col].min()

            asint = df[col].fillna(0).astype(np.int64)
            result = (df[col] - asint)
            result = result.sum()
            if result > -0.01 and result < 0.01:
                IsInt = True

            if IsInt:
                if minValue >= 0:
                    if maxValue < np.iinfo(np.uint8).max:
                        df[col] = df[col].astype(np.uint8)
                    elif maxValue < np.iinfo(np.uint16).max:
                        df[col] = df[col].astype(np.uint16)
                    elif maxValue < np.iinfo(np.uint32).max:
                        df[col] = df[col].astype(np.uint32)
                    else:
                        df[col] = df[col].astype(np.uint64)
                else:
                    if minValue > np.iinfo(np.int8).min and maxValue < np.iinfo(np.int8).max:
                        df[col] = df[col].astype(np.int8)
                    elif minValue > np.iinfo(np.int16).min and maxValue < np.iinfo(np.int16).max:
                        df[col] = df[col].astype(np.int16)
                    elif minValue > np.iinfo(np.int32).min and maxValue < np.iinfo(np.int32).max:
                        df[col] = df[col].astype(np.int32)
                    elif minValue > np.iinfo(np.int64).min and maxValue < np.iinfo(np.int64).max:
                        df[col] = df[col].astype(np.int64)

            else:
                if minValue > np.finfo(np.float16).min and maxValue < np.finfo(np.float16).max:
                    df[col] = df[col].astype(np.float16)
                elif minValue > np.finfo(np.float32).min and maxValue < np.finfo(np.float32).max:
                    df[col] = df[col].astype(np.float32)
                elif minValue > np.finfo(np.float64).min and maxValue < np.finfo(np.float64).max:
                    df[col] = df[col].astype(np.float64)   

            if(verbose==True):
                print("Column: ", col, ", converted dtype: ",df[col].dtype)
   
    if(dispInfo):
        processedSize = df.memory_usage().sum()
        print("Shrinked by ", "{0:.2f}".format(((initialSize - processedSize) / (1024**2))), " MB, to ", "{0:.2f}".format(100*processedSize/initialSize), "% of original size")
```
----------------------------------------------------------------------------------------------------------------