# Setup-Creator:

</br>

```ruby
Compiler    : Delphi10 Seattle, 10.1 Berlin, 10.2 Tokyo, 10.3 Rio, 10.4 Sydney, 11 Alexandria, 12 Athens
Components  : None
Discription : Setup and Uinstall Creator for Delphi Projects
Last Update : 09/2025
License     : Freeware
```

</br>

A Setup Program is specialized Software used to install and configure other applications, Operating Systems, or Hardware drivers on a Computer. It often guides the user through a process with dialog boxes to copy all necessary files, make configuration settings, and prepare the Program for use.

Using this example, you can create your own setup.exe and uninstall.exe for your project. You can integrate and install as many files as needed. Registry entries will be made so that the uninstaller can subsequently find and uninstall all or individual files.

</br>

![setup creator](https://github.com/user-attachments/assets/a5a55469-b80a-4291-bed5-f997d8cc0ea7)

</br>

### Instructions:
* To integrate your own files into the setup, you first need to create the *.res files.

```pascal
program CustomInstaller;

uses
  Forms,
  fSetup in 'fSetup.pas' {Form1};

{$R *.res}
{$R AppFiles.res}      <-- Install Files
{$R AppResources.res}  <-- Media Files

begin
  Application.Initialize;
  Application.CreateForm(TForm1, Form1);
  Application.Run;
end.
```

* To do this, you need the "brcc32.exe", which you can find in your Delphi Program under the "..\bin\brcc32.exe" folder. Copy this file into your project folder.
* Next, you need to specify and save the files you want to include in your setup program in the *.rc file.


### {$R AppFiles.res}
```pascal
MyApp    RCDATA "MyApp.exe"
dllA     RCDATA "A.DLL"
dllB     RCDATA "B.DLL"
HelpFile RCDATA "MyApp.chm"
Uninstaller RCDATA "Uninstall.exe"
```

### {$R AppResources.res}
```pascal
MyAppImage    RCDATA "MyApp.jpg"
SetupIcon     RCDATA "SetupIcon.jpg"
```

* Now Drag and Drop the "*.rc" File onto the "brcc32.exe" and the "*.res" file will be created.
* Now you can integrate and use the created files in your project.





