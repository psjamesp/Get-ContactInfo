# Style Guide

The previous header should include your chapter name. The name of this chapter is "Style Guide".

## Writing Style

This is an example of a section in a Chapter. The title for this section is "Writing Style".

Per Jeff Hicks as referenced in [Issue 12](https://github.com/mikefrobbins/powershell-conference-book/issues/12), write from the reader's point of view:

    "You will need to make sure the -ErrorAction parameter is set to Stop."

Although I think it is perfectly acceptable to write about steps you are doing to contrast what the reader is doing.

    "In my network, I had to give the network adapter a static IP address. You should be able to use DHCP."

## Images

This is an example of another section in this chapter. The name of this section is "Images".

Images should be the exception and not the rule. Be sure to read the "Text Based Output" section of this chapter for clarification on what the previous sentence means and to prevent having to redo the output of the commands for your chapter.

Per Jeff Hicks recommendation in [Issue 11](https://github.com/mikefrobbins/powershell-conference-book/issues/11), the naming convention of images should be lastname-some-brief-description.png. Use all lower case. Images should be high resolution.

### Text Based Output

This is an example of a subsection of the Images section. The name of this subsection is "Text Based Output".

Provide the reader with the text based output of the command instead of an image of the output. When doing this, a console width of 80 seems to work best so the output is already wrapped instead of having Leanpub mangle it.

![](images/lastname-console-width.png)

Use a font size of 12 with all of the other settings set to the default.

![](images/lastname-console-font.png)

If an image or screenshot of output from a command is provided in your chapter instead of the text based output, you will be asked to remove the image and replace it with the text based output during the review process.

If you give the image a name within the square brackets, it will show up at the bottom of the image in the ebook that Leanpub produces, although it doesn't show up in the preview of most markdown editors.

#### PowerShell Version

This is an example of a second level subsection. It is a subsection of the "Text Based Output" subsection. Try to limit your writing to no more sublevels than this.

You can use Windows PowerShell or PowerShell Core, but try to stick with the same one as well as the same look and feel throughout your chapter unless you're specifically showing the differences between the two.

The syntax for opening and closing blocks of code is three tildes. Turn off line numbers for the output and place the copied output from the console in another code block. This may not display properly within the preview of your markdown editor, but it will show line number for the code and no line numbers for the output in the book.

```powershell
PS C:\> Get-Command -Name Get-Process, Get-Service
```
{line-numbers=off}
```powershell
CommandType     Name                          Version    Source
-----------     ----                          -------    ------
Cmdlet          Get-Process                   3.1.0.0    Microsof...
Cmdlet          Get-Service                   3.1.0.0    Microsof...
...

PS C:\>
```

You may want to limit the output in the book and mention it to the reader if it's excessively long. You should also not be afraid to adjust output so that a given line is less than 80 characters long. In the example above extra spaces were removed to shorten up the line length to avoid ugly wrapping.

#### Inline Code

As referenced in [Issue 12](https://github.com/mikefrobbins/powershell-conference-book/issues/12), Commands names should be written in proper case when written within a paragraph such as `Get-Service` and they should be styled as inline code by wrapping the command name in backticks.


## Table of Contents

This is an example of a another section of this chapter. It is at the same level as images. The name of this section is "Table of Contents".

The benefit of using sections and subsections in a chapter is that they are used to automatically create the table of contents for the book.
