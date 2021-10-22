One of the themes of .NET 6 is writing less boilerplate code and helping developers focus on their code instead of ceremony coming with the programming language and the framework. The file scoped namespaces feature is one of C# 10 features that we can consider under that theme.

Visual Studio offers a fix to convert a block-scoped namespace to a file scoped namespace. However, you can only apply it for the current document. Therefore, it's not possible to fix all occurrences in a project or a solution.

![visual-studio-convert-to-file-scoped-fix-dialog](https://az718566.vo.msecnd.net/uploads/2021/10/22/vs-fix-dialog.png)

On the other hand, it is possible to convert a file scoped namespace to a block-scoped namespace and fix all occurrences in a project or a solution.

![](https://az718566.vo.msecnd.net/uploads/2021/10/22/block-scoped-refactoring.png)

This is how the refactoring/fix pair works. The default namespace declaration preference is block scoped. So, Visual Studio provides refactoring for converting file scoped namespaces to block-scoped namespaces.

There are a couple of ways to change the default namespace declaration preference.

## .editorconfig File

If your project or solution contains a `.editorconfig` file, you can add the following line and make the file scoped namespaces the default preference.

```bash
csharp_style_namespace_declarations = file_scoped
```

After adding the line, Visual Studio will offer the "Convert to file-scoped namespace" refactoring and you can fix all occurrences in a project or a solution.

![](https://az718566.vo.msecnd.net/uploads/2021/10/22/convert-to-namespace-refactoring.png)

## Changing Visual Studio Code Style Preference
If your project doesn't contain a `.editorconfig` file or you look for an alternative way, you can change the code-style preference in Visual Studio globally so that namespace preference applies to all projects that you work on. To change the preference globally, open the Options window(**Tools=>Options**). Navigate to the C# code style preferences by clicking **Text Editor=>C#=>Code Style** and then change the related preference. 

![](https://az718566.vo.msecnd.net/uploads/2021/10/22/visual-studio-namespace-preference.png)

I hope this post helps you to adapt to the new features coming with C# 10. 