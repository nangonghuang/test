

https://git-scm.com/docs/gitignore

匹配规则：

1. 空白行什么文件都不匹配，因此为了方便阅读可以作为分隔符
2. 以#开头的行是注释。
3. 尾部空格会被忽略，除非它们和“\”一起使用
4. 
5. 如果模式以/结尾，表示一个目录和它的子目录，不会匹配文件或者快捷方式
6. 如果模式不包含/，git会把它作为一个Shell glob规则然后从.gitignore文件的目录下开始匹配
7. 如果包含/，通配符不会匹配/，比如 "Documentation/*.html" 匹配 "Documentation/git.html" 不匹配 "Documentation/ppc/ppc.html" or或者 "tools/perf/Documentation/perf.html".
8. /开头的行，匹配路径的开始，比如 "/*.c" 匹配 "cat-file.c" 而不匹配 "mozilla-sha1/sha1.c".

**有特殊的含义：

 1. \*\*后接斜线表示匹配所有目录 。 "\*\*/foo" 匹配所有地方的 "foo" 文件,和 "foo"一样。 "**/foo/bar" 匹配任意地方的foo目录下的bar文件或者目录
 2. 结尾的/\*\* 匹配里面的所有东西。比如， "abc/**" 匹配相对于.gitignore文件位置的abc目录下的所有文件，递归到无限深度。
 3. /\*\*/ 匹配0或者多个目录，比如"a/**/b" 匹配 "a/b", "a/x/b", "a/x/y/b" 等等.
 4. 其他连续的*号被视为无效




PATTERN FORMAT

    A blank line matches no files, so it can serve as a separator for readability.

    A line starting with # serves as a comment. Put a backslash ("\") in front of the first hash for patterns that begin with a hash.

    Trailing spaces are ignored unless they are quoted with backslash ("\").

    An optional prefix "!" which negates the pattern; any matching file excluded by a previous pattern will become included again. It is not possible to re-include a file if a parent directory of that file is excluded. Git doesn’t list excluded directories for performance reasons, so any patterns on contained files have no effect, no matter where they are defined. Put a backslash ("\") in front of the first "!" for patterns that begin with a literal "!", for example, "\!important!.txt".

    If the pattern ends with a slash, it is removed for the purpose of the following description, but it would only find a match with a directory. In other words, foo/ will match a directory foo and paths underneath it, but will not match a regular file or a symbolic link foo (this is consistent with the way how pathspec works in general in Git).

    If the pattern does not contain a slash /, Git treats it as a shell glob pattern and checks for a match against the pathname relative to the location of the .gitignore file (relative to the toplevel of the work tree if not from a .gitignore file).

    Otherwise, Git treats the pattern as a shell glob suitable for consumption by fnmatch(3) with the FNM_PATHNAME flag: wildcards in the pattern will not match a / in the pathname. For example, "Documentation/*.html" matches "Documentation/git.html" but not "Documentation/ppc/ppc.html" or "tools/perf/Documentation/perf.html".

    A leading slash matches the beginning of the pathname. For example, "/*.c" matches "cat-file.c" but not "mozilla-sha1/sha1.c".

Two consecutive asterisks ("**") in patterns matched against full pathname may have special meaning:

    A leading "**" followed by a slash means match in all directories. For example, "**/foo" matches file or directory "foo" anywhere, the same as pattern "foo". "**/foo/bar" matches file or directory "bar" anywhere that is directly under directory "foo".

    A trailing "/**" matches everything inside. For example, "abc/**" matches all files inside directory "abc", relative to the location of the .gitignore file, with infinite depth.

    A slash followed by two consecutive asterisks then a slash matches zero or more directories. For example, "a/**/b" matches "a/b", "a/x/b", "a/x/y/b" and so on.

    Other consecutive asterisks are considered invalid.

