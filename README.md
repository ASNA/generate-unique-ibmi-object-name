## Generate a unique IBM i object id 

The `shortid` [Nuget package](https://www.nuget.org/packages/shortid) generates unique, short ids. Out of the box is it is effective, but its default doesn't generate unique ids that work as IBM i object names. 

This wrapper function adds the constraints needed to generate a unique IBM i object name:

```
Using ShortId 
Using ShortId.Configuration 

...

BegFunc GenerateUniqueId Type(*Char) Len(10) 
    // ShortId class comes from this nuget package: https://www.nuget.org/packages/shortid

    DclFld Id Type(*Char) Len(10)

    DclFld Options Type(GenerationOptions) New()
    Options.Length = 10
    // ShortId needs at least 50 unique characters. 
    ShortId.SetCharacters('ABCDEFGHIJKLMNOPQRSTUVWXYZ01234567890$#@abcdefghijklmnopqrstuvwxyz')

    Id = ShortId.Generate(Options).ToUpper()
    // Result cannot contain _, -, or start with a digit
    DoWhile Id.Contains('_') OR Id.Contains('-') OR Regex.IsMatch(Id, "^\d") 
        Id = ShortId.Generate(Options)
    EndDo             

    LeaveSr Id
EndFunc
```    
Every call to GenerateUniqueId() returns a unique 10-character object id. To ensure the added constraints don't affect ShortId's unique random capabilities, I used this repo to generate 1m ids and check each for uniqueness. 

The test took a long time (each unique id is stashed in a string collection -- that gets very large before the test is completed.) Despite taking a long time, the GenerateUniqueId() did indeed return 1m unique object ideas -- no duplicates. 

### To use the ShortID package in Visual RPG

Download the `shortid` package from the "Download package" link on the right side of [this page](https://www.nuget.org/packages/shortid).

Despite the download's `.nupkg` extension, Nuget packages are zip files. Use your favorite zip utility to unzip the package. After unzipping, set a reference to this file in the package 

    \lib\netstandard1.0\netstandard1.0\shortid.dll

in your Visual Studio project. Then, add these two `Using` statements:

```
Using ShortId 
Using ShortId.Configuration 
```

The `GenerateUniqueId` function (as shown above) compiles successfully after doing that setup. 