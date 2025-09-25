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

* Now Drag and Drop the "*.rc" File onto the "brcc32.exe" and the "*.res" file will be created. The more files you include as RCDATA, the larger the "*.res" File in your Project will be.
* Now you can integrate and use the created files in your project.

* This part of the code registers your program's links in the registry.
```pascal
try
         reg.RootKey := HKEY_CURRENT_USER;
         reg.OpenKey('Software\MicroSoft\Windows\CurrentVersion\Explorer\Shell Folders', false);

         case ShortcutFolder of
            sfCustom: { Do nothing, it has been covered above! };
            sfStartMenu: sDirectory := reg.ReadString('Start Menu') + '\' + sShortcutFolder;
            sfPrograms: sDirectory := reg.ReadString('Programs') + '\' + sShortcutFolder;
            sfDesktop: sDirectory := reg.ReadString('Desktop') + '\' + sShortcutFolder;
         end;
         
      finally
         reg.Free
end;
```

* To let the Windows system know which program is installed on your system, your program's key is written to the registry here. This section is important for uninstalling or updating your program.

```pascal
function GetProgramFilesDirectory: string;
var
   reg: TRegistry;
begin
   reg := TRegistry.Create;
   try
      reg.RootKey := HKEY_LOCAL_MACHINE;
      reg.OpenKey('\SOFTWARE\Microsoft\Windows\CurrentVersion', True);
      result := reg.ReadString('ProgramFilesDir');
   finally
      reg.Free
   end;
   { Append a backslash at the end (if not present already }
   if (result <> '') and (result[Length(result)] <> '\') then
      result := result + '\';
end;
```

* Here, you'll need to add all the files you specified as names in the RCDATA file so the setup program can install them into the system. This also applies to the uninstaller.

```pascal
procedure TForm1.DoInstall(Sender: TObject);
begin
   { Hide "Install button }
   btnInstall.Visible := false;

   { read target folder from the appropraite edit box text }
   sInstallDir := edTargetFolder.Text;
   { Create the folder full path } 
   ForceDirectories(sInstallDir);

   { Extract the application files }
	DoExtractResource('MyApp', sInstallDir + 'MyApp.exe');
	DoExtractResource('dllA', sInstallDir + 'A.DLL');
	DoExtractResource('dllB', sInstallDir + 'B.DLL');
	DoExtractResource('Helpfile', sInstallDir + 'MyApp.chm');

   { Extract the uninstaller executable as well. }
	DoExtractResource('Uninstaller', sInstallDir + 'Uninstall.exe');

   { Create the shortcuts as required }
   CreateShortcuts;

   { Save the installation directory in Windows registry }
   SaveInstallationDirInRegistry;

   { Register the uninstaller application with the "Add / remove programs" in Control Panel }
   RegisterUninstallerWithControlPanel; 
end;
```

* In this section you can determine which of your files that are installed into the system will be linked on the desktop.

```pascal
procedure TForm1.CreateShortcuts;
begin

   { shortcut in "programs" - ALWAYS }
   CreateShortCut(sInstallDir + 'MyApp.exe',
                  '',
                  'MyApp Description', 'MyApp.lnk',
                  sfPrograms,
                  'MyApp',
                  '');

   { shortcut on desktop }
   if chShortcutOnDesktop.Checked then
      CreateShortCut(sInstallDir + 'MyApp.exe',
                     '',
                     'MyApp Description', 'MyApp.lnk',
                     sfDesktop,
                     '',
                     '');

   { shortcut in start menu }
   if chShortcutInStartMenu.Checked then
      CreateShortCut(sInstallDir + 'MyApp.exe',
                     '',
                     'MyApp Description', 'MyApp.lnk',
                     sfStartMenu,
                     '',
                     '');

   { a shortcut for the application help file in "programs" - ALWAYS }
   CreateShortCut(sInstallDir + 'MyApp.chm',
                  '',
                  'MyApp HTML help', 'MyAppHelp.lnk',
                  sfPrograms,
                  'MyApp',
                  '');

   { Finally, a shortcut for Uninstaller in "programs" - ALWAYS }
   CreateShortCut(sInstallDir + 'Uninstall.exe',
                  '',
                  'Uninstall MyApp', 'MyAppUninstaller.lnk',
                  sfPrograms,
                  'MyApp',
                  '');

end;
```

</br>

### Uninstall Program:
* Open the Uninstallation Project.
* To successfully remove your installed program from the system, you must enter the registry key that you used during installation.
```pascal
procedure TfrmUninstall.LoadInstallationDirFromRegistry;
var
   reg: TRegistry;
begin
   reg := TRegistry.Create;
   try
      reg.RootKey := HKEY_CURRENT_USER;
      reg.OpenKey('Software\MyCompany\MyApp', true);
      sInstallDir := reg.ReadString('Installation directory');
   finally
      reg.Free
   end;
end;
```

* Here you can specify which categories should be uninstalled.
```pascal
procedure TfrmUninstall.btnUninstallClick(Sender: TObject);
begin

   { Find out where the application has been installed }
   LoadInstallationDirFromRegistry;

   { Remove the last backslash from the installation folder name }
   Delete(sInstallDir, length(sInstallDir), 1);

   { Delete the created shortcuts }
   DeleteShortcuts;

   { Delete the Windows registry keys we created at installation }
   DeleteRegistryKeys;

   { "Unregister" the uninstaller app from the "Add/Remove programs" control panel section }
   UnregisterUninstaller;

   { Delete the installation folder }
   if DirectoryExists(sInstallDir) then
      SH_DeleteFiles(sInstallDir);

   MessageDlg('MyApp has been successfully removed from your computer.', mtInformation, [mbOK], 0);

   Close;
end;
```




