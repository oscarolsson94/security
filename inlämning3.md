## Exploit

1. Visit the url of the singleFlagPage `/flag`
2. To get access to the secret file containing API-keys, enter "../" in the URL after the parameter `?name=`. Like this: `/flag?name=../keys.xml`
3. Press enter, and u will see the contents of the secret XML-file on your screen.

## Vulnerability

The vulnerable lines of code are in the `singleFlagPage` method:
```
private static void singleFlagPage(Context context) throws IOException {
        String flagName = context.queryParam("name");
        Path path = Path.of("flags/" + flagName);
        String svg = Files.readString(path);
        context.contentType("image/svg+xml; charset=UTF-8");
        context.result(svg);
    }
```
We are not checking the users input before actually reading the file. Therefore we allow the user full access to our entire file system. If the user decides to enter `../` either once, or even a couple of times, they would be able to see files in folders where we do not want the user to have access to.

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

*Note: This solution only limits the user to save a file in a specific folder. Files in that folder can still be overwritten if the user chooses the same name as an already existing one when saving a file. To fix this issue, we would have to check if a file with the entered name already exists in the folder before writing to the file.* 
