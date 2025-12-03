+++
title = "Shell"
date = "2025-03-31"
draft = false
cover = "images/cover/shell.png"
#tags = [""]
showFullContent = false
hideComments = true

#for SEO
#keywords = [""]
#description = ""
+++
[Github](https://github.com/asdiAdi/codecrafters-shell-csharp)

Challenge attempt for creating my own [Shell](https://app.codecrafters.io/courses/shell/overview).

<!--more-->

{{< details summary="TLDR" >}}
- Used C#.
- Implemented `echo`, `type`, `exit`, `pwd`, `cd` etc.
{{< /details >}}

In this guided project, I will attempt to build my own  [POSIX compliant](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html) shell.
This is an attempt in the challenge "[Build your own Shell](https://app.codecrafters.io/courses/shell/overview)" from codecrafters.
The project is designed to be refactored each step of the way. I will paste the code that passes the tests at each stage, but I won't update the pasted code after refactoring.

## Base
### Repository Setup
I will be using C# for this attempt.

### Print a prompt
Wait for user input with `$ `
```c#
Console.Write("$ ");
```

### Handle invalid commands
Treat all input as invalid
```c#
Console.WriteLine($"{input}: command not found");
```

### REPL
Implement a [REPL (Read-Eval-Print Loop)](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) using a loop.
```c#
do {
...
} while(true)
```

### The exit builtin
[exit](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#exit) - cause the shell to exit
Implement by checking the command name.
```c#
var command = splitInput[0];
...
switch (command) {
case "exit":
    if (!string.IsNullOrEmpty(operand))
    {
        return 0;
    }
```

### The echo builtin
[echo](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/echo.html) - write arguments to standard output
Implement by checking the command name.
```c#
case "echo":
    Console.WriteLine(operand);
    break;
```

### The type builtin: builtins
The type builtin is used to determine how a command would be interpreted if used.
```c#
private static readonly string[] BuiltInCommands = ["echo", "type", "exit"];
...
case "type":
    if (BuiltInCommands.Contains(operand))
    {
        Console.WriteLine($"{operand} is a shell builtin");
    }
    else
    {
        Console.WriteLine($"{operand}: not found");
    }
    break;
```

### The type builtin: executable files
[PATH](https://en.wikipedia.org/wiki/PATH_(variable)) is an environment variable that specifies a set of directories where executable programs are located. 
```c#
var paths = Environment.GetEnvironmentVariable("PATH");
bool isFound = false;
if (!string.IsNullOrEmpty(paths))
{
    foreach (var path in paths.Split(':'))
    {
        var joinedPath = Path.Join(path, operand);
        if (!File.Exists(joinedPath)) continue;
        isFound = true;
        Console.WriteLine($"{operand} is {joinedPath}");
        break;
    }
}
```

### Run a program
Add support for running external programs.
```c#
string? FindPath(string file)
{
    if (string.IsNullOrEmpty(paths)) return null;
    foreach (var path in paths.Split(':'))
    {
        var joinedPath = Path.Join(path, file);
        if (!File.Exists(joinedPath)) continue;
        return path;
    }

    return null;
}
...
else if (!string.IsNullOrEmpty(commandPath))
{
    Process.Start(command, operand);
}
```

## Extension: Navigation

### The pwd builtin
[pwd](https://en.wikipedia.org/wiki/Pwd) stands for "print working directory".
```c#
case "pwd":
    Console.WriteLine(Directory.GetCurrentDirectory());
    break;
```

### The cd builtin: Absolute paths
The `cd` command is used to change the current working directory.
```c#
case "cd":
    if (Path.Exists(operand))
        Directory.SetCurrentDirectory(operand);
    else
        Console.WriteLine($"cd: {operand}: No such file or directory");
    break;
```

### The cd builtin: Relative paths
Relative paths are `./`, `../`, `./dir`.
```c#
if (newDirectory.StartsWith("./"))
{
    newDirectory = Path.Join(currentDirectory, operand);
}
else if (newDirectory.StartsWith("../"))
{
    while (newDirectory.StartsWith("../"))
    {
        currentDirectory = Path.GetDirectoryName(currentDirectory) ?? string.Empty;
        newDirectory = newDirectory[3..];
    }
    newDirectory = currentDirectory;
}
```

### The cd builtin: Home directory
The `~` character, which stands for the user's home directory.
```c#
else if (newDirectory.StartsWith("~"))
{
    newDirectory = Environment.GetEnvironmentVariable("HOME") ?? "";
}
```

## Extension: Quoting

### Single quotes
Enclosing characters in single quotes preserves the literal value of each character within the quotes.
```c#
var arg =
    (operand.FirstOrDefault() ==
     '\'') switch
    {
        true => Regex.Matches(operand.Replace("''", ""), "'(.*?)'")
            .Select(m => m.Groups[1].Value)
            .ToArray(),

        false => Regex.Split(input, "\\s+").Skip(1).ToArray()
    };
```

### Double quotes
Enclosing characters in double quotes preserves the literal value of each character within the quotes except `\`, the backslash retains its special meaning when followed by `\`, `$`, `"` or newline.
```c#
else if (operand.FirstOrDefault() ==
         '\"')
{
    arg = Regex.Matches(operand.Replace("\"\"", ""), "\"(.*?)\"")
        .Select(m => m.Groups[1].Value)
        .ToArray();
}
```

### Backlash outside quotes
A [non-quoted backslash](https://www.gnu.org/software/bash/manual/bash.html#Escape-Character) `\` is treated as an escape character. It preserves the literal value of the next character. 
```c#
else
{
    arg = Regex.Matches(operand, "([^\\\\\\s]+[^\\\\\\s]\\s|[^\\\\\\s]+)|(\\\\)(.)")
        .Select(m => !string.IsNullOrEmpty(m.Groups[1].Value)
            ? m.Groups[1].Value
            : m.Groups[3].Value
        )
        .ToArray();
}
```

### Backslash within single quotes
Enclosing characters in single quotes `'` preserves the literal value of each character within the quotes, even backslashes.
No code changes needed.

### Backslash within double quotes 
Enclosing backslashes within double quotes `"` preserves the special meaning of the backslash, only when it is followed by `\`, `$`, `"` or newline.
```c#
else if (parseMode == ParseMode.DoubleQuote)
...
if (operand[i] == '\"')
{
    parseMode = ParseMode.Default;
    if (arg == string.Empty) continue;
    args.Add(arg);
    arg = string.Empty;
}
```

### Executing a quoted executable
Implementing support for executing a quoted executable.
```c#
List<string> parsedArguments = ParseArgumentsFromInput(input);
if (parsedArguments.Count == 0) continue;
var command = parsedArguments[0].Replace("\'", "\\'");
string? commandPath = FindPath(command);
...
ExternalCommandRunner(command, arguments.Where(a => a != " " && a != "").ToList());
```

## Navigation

### Builtin completion
Implement autocomplete when pressing `<TAB>`.
Input: `ech` Output: `echo`
Input: `exi` Output: `exit`

```c#
else if (key.Key == ConsoleKey.Tab)
{
    List<string> args = ParseArgumentsFromInput(input.ToString());
    if (args.Count != 1) continue;
    string command = args[0];
    string[] match = BuiltInCommands.Where(str => str.StartsWith(command)).ToArray();
    if (match.Length == 1)
    {
        Console.Write(match[0].Substring(command.Length) + " ");
        input.Append(match[0].Substring(command.Length) + " ");
    }
}
```

### Completion with arguments
Shell should correctly handle the subsequent arguments that the user types.
No Code Changes Needed.

### Missing completions
When the user types a command that is not a known builtin and presses `<TAB>`, your shell should not attempt to autocomplete it.
```c#
if (match.Length == 1)
{
    Console.Write(match[0].Substring(command.Length) + " ");
    input.Append(match[0].Substring(command.Length) + " ");
} else if (match.Length == 0)
{
    Console.Write('\a');
}
```

### Executable completion
Include external executable files in the user's `PATH`.
If you have a command `custom_executable` in the path and type `custom` and press `<TAB>`, the shell should complete that to `custom_executable`.
```c#
private static string[] FindCommands(string command)
{
    string[] commands = BuiltInCommands.Where(str => str.StartsWith(command)).ToArray();

    if (string.IsNullOrEmpty(PATHS)) return null;
    foreach (var path in PATHS.Split(':'))
    {
        if (!Directory.Exists(path)) continue;
        string[] exactMatch = Directory.GetFiles(path, command);
        string[] patternMatch = Directory.GetFiles(path, $"{command}*");
        string[] paths = exactMatch.Concat(patternMatch).ToArray();
        
        string[] _commands = paths.Select(p => p.Substring(p.IndexOf(command))
        ).ToArray();
        

        if (_commands.Length > 0)
            commands = commands.Concat(_commands).ToArray();
    }

    return commands.Distinct().ToArray();
}
```
### Multiple completions
On the first TAB press, just ring a bell. (`\a` rings the bell)
On the second TAB press, print all the matching executables separated by 2 spaces, on the next line, and follow it with the prompt on a new line.
```c#
else // multiple matches
{
    if (!isFirstTab)
    {
        Console.WriteLine();
        Console.Write($"{string.Join("  ", match)}");
        Console.WriteLine();
        Console.Write($"$ {command}");
        isFirstTab = true;
    }
    else
    {
        Console.Write('\a');
        isFirstTab = false;
    }
}
```

### Partial completions
When the user types a partial command and presses the Tab key, your shell should attempt to complete the command name.
if `xyz_foo`, `xyz_foo_bar`, and `xyz_foo_bar_baz` are all available executables and the user types `xyz_` and presses tab, then your shell should complete the command to `xyz_foo`.

```c#
List<string> match = FindCommands(command);
List<string> filteredMatch = new List<string>(match);
for (int i = 1; i < filteredMatch.Count; i++)
{
    for (int j = 0; j < i; j++)
    {
        if (filteredMatch[i].StartsWith(filteredMatch[j]))
        {
            filteredMatch.RemoveAt(i);
            i--;
            break;
        }
    }
}

if (filteredMatch.Count == 1)
{
    string output = match[0].Substring(command.Length) + (match.Count > filteredMatch.Count
        ? ""
        : " ");
    Console.Write(output);
    input.Append(output);
}
```

## Redirection

### Redirect stdout
Implement support for [redirecting the output](https://www.gnu.org/software/bash/manual/bash.html#Redirecting-Output) of a command to a file.
The `1>` operator is used to redirect the output of a command to a file.
```c#
private static void stdout(string text, string? target = null)
{
    if (target == null)
    {
        Console.WriteLine(text);
    }
    else
    {
        Directory.CreateDirectory(Path.GetDirectoryName(target));
        File.Create(target).Dispose();
        File.AppendAllText(target, text + Environment.NewLine);
    }
}
//replace all Console.Writelines with stdout eg:
case "echo":
    stdout(operand, target);
    break;
```

### Redirect stderr
Implement support for redirecting the standard error of a command to a file.
The `2>` operator is used to redirect the standard error of a command to a file.
```c#
public void Error(string text)
if (isStdErr && target != null)
            {
                Directory.CreateDirectory(Path.GetDirectoryName(target));
                File.Create(target).Dispose();
            }
...
{
    if (isStdErr && target != null)
        File.AppendAllText(target, text + Environment.NewLine);
    else
        Console.WriteLine(text);
}
```

### Append stdout
Implement support for appending the output of a command to a file.
The `1>>` operator is used to append the output of a command to a file. 

```c#
public class StdOut(bool isStdOut, bool isStdErr, bool isStdAppend, string? target)
{
    public void Write(string text)
    {
        if ((isStdOut || isStdAppend) && target != null)
        {
            Directory.CreateDirectory(Path.GetDirectoryName(target));
            if (!isStdAppend)
                File.Create(target).Dispose();
            File.AppendAllText(target, text + Environment.NewLine);
```

### Append stderr
Implement support for appending the standard error of a command to a file.
The `2>>` operator is used to append the standard error of a command to a file.
```c#
if ((isStdErr || isStdErrAppend) && target != null)
{
    Directory.CreateDirectory(Path.GetDirectoryName(target));
    if (!File.Exists(target) || !isStdErrAppend)
        File.Create(target).Dispose();
}
```

## Conclusion
It was a fun guided project. I got bored at the end that's why the navigation implementation is shitty.