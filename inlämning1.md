## Inlämning 1

## Exploit

1. To get access to files you should not be able to, enter "../" when you are about to save a new file to the app.
2. Save a file with the same path and name as an already existing file, for example: `../secrets/passwords.txt`
3. The new file you saved has now overwritten the already existing one, and the old file and its contents are now deleted.

## Vulnerability

The vulnerable lines of code are in the `publish` method:
```
private static void publish(Context context) throws IOException {
        String filename = context.formParam("filename");
        String text = context.formParam("text");

        Path path = Path.of("stories/" + filename);
        Files.writeString(path, text);   
        ...
    }
```
We are not checking the users input before actually writing to the file. Therefore we allow the user full access to our entire file system. If the user decides to enter `../` either once, or even a couple of times, they would be able to see, create, and overwrite files in folders where we do not want the user to have access to.

## Fix

To fix this vulnerability, we make use of `toAbsolutePath` followed by `normalize` to get the full path of the user input without extra `../`. We also grab the actual path of the "stories" folder.
```
        String filename = context.formParam("filename");
        String text = context.formParam("text");

        Path path = Path.of("stories/" + filename).toAbsolutePath().normalize();
        Path storyFolder = Path.of("stories").toRealPath();   
        ...
```
Before writing anything to the file, me make sure to compare the path of the folder which we want the user to have access to, with the path entered by the users. If the path does not match, we throw an error telling the user they cannot save files outside the story folder.
```
        if (!path.startsWith(storyFolder)) {
            context.status(403);
            context.result("You cannot save files outside the story folder.");
            return;
        }

        Files.writeString(path, text);

    }
```
We have now protected ourselves against malicious users, who may try to either retrieve or delete our assets. We only allow the user to save files in the story-folder.  

*Note: This solution only limits the user to save a file in a specific folder. Files in that folder can still be overwritten if the users chooses the same name when saving a file. To fix this issue, we would have to check if a file with the entered name already exists in the folder before writing to the file.* 
