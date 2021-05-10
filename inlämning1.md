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

*Note: This solution only limits the user to save a file in a specific folder. Files in that folder can still be overwritten if the user chooses the same name as an already existing one when saving a file. To fix this issue, we would have to check if a file with the entered name already exists in the folder before writing to the file.* 



## Exploit

1. When adding a new note, enter `abc'); DROP TABLE user --`
2. The `'` will end the SQL-string, and allow for another SQL-command to be entered.
3. `--` makes everything after the user-string count as a comment.
4. You have now deleted the entire user-table from the database.

## Vulnerability

The vulnerable lines of code are at the post endpoint to `/add` :
```
 app.post("/add", context -> {
            int userId = context.sessionAttribute("userId");
            String noteText = context.formParam("note");
            String dateTimeString = LocalDateTime.now().toString();

            try (Connection c = db.getConnection()) {
                Statement s = c.createStatement();
                s.executeUpdate(
                    "INSERT INTO note(user_id, datetime, text) VALUES (" +
                    userId + ", '" + dateTimeString + "', '" + noteText + "')"
                );
            }
            context.redirect("/");
        });
```
The SQL-string is using string concatenation without doing any kind of validation before executing the DB-query. This is very dangerous, and should never be used. The user is able to invoke any kind of SQL-statement by simply ending the original string with a `'`, and then continuing with entering their own expression. User input is directly injected into the SQL-query, and by ending the input with `--`, the user can avoid SQL-exceptions since this makes SQL interpret all the following as a simple comment.

## Fix
The fix to this exploit is fairly easy, as we do not need to implement our own custom solution. Instead of using string concatination to build our SQL-string, we simply make use of the already existing `PreparedStatement` class. Which has built in validation to combat SQL-injections, and does not allow the user to end an SQL-statement with `'`. The solution looks like this:
```
app.post("/add", context -> {
            int userId = context.sessionAttribute("userId");
            String noteText = context.formParam("note");
            String dateTimeString = LocalDateTime.now().toString();

            try (Connection c = db.getConnection()) {
                // FIXED: This fixes exploit #2.
                PreparedStatement s = c.prepareStatement(
                    "INSERT INTO note(user_id, datetime, text) VALUES (?, ?, ?)"
                );
                s.setInt(1, userId);
                s.setString(2, dateTimeString);
                s.setString(3, noteText);
                s.executeUpdate();
            }
            context.redirect("/");
        });
```

We have now protected ourselves against malicious SQL-injections. The user input is longer directly injected into the SQL-string by string concatination, but instead through the `PreparedStatement` class. 
