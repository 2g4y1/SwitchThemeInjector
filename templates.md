# Templates
To keep the code simple this tool uses templates and patches to edit layouts.
A template tells the program how to apply a background image to an SZS, while a layout patch can change to position of the UI elements. Both are Json files. \
All the default templates are hardcoded but as of V 3.0 you can place a file called 
ExtraTemplates.json in the same folder of the executable to load more templates. \
Version 3.2 added support for layout patches, put them in the "Layouts" folder.
All extra templates must be in the same file, while each layout patch is a single file, 
this is because an SZS can have only one template that is selected automatically but it can have multiple layout patches because the user can choose which one apply.
Compatibilty is automatically checked when an SZS is loaded.
\
This file explains how does patching work and how to build a custom template.
# Layout patching
It's easy to think that adding a background is just a matter of replacing the bg
image, and it's the way we would do it if there was already a background picture, but unfortunately in most layouts there isn't, so we need to add one by ourselves.
These are the steps the injector follows when creating a theme :
1) Inject an image in the BNTX. We don't fully understand the BNTX format so we 
can't add a new image yet, for now we need to replace an image that's already in there
2) Patch the layout that renders the background : this process adds an entry in the
texture list for the texture we replaced, adds a new material and a new image pane for the background.
3) Patch all the layouts that use the image we replaced to use another similar image 
4) If the user selected a layout patch apply it.

So our patch format must store all the information needed for this process.
# The templates (also referred as patch templates)
Here we take a look at the values in the format.
For reference you can download the default patches as a json from this repo (These are hardcoded, you don't need them in your own json).\
First the Metadata : \
`FirmName`, `TemplateName`, `szsName` and `TitleId` are the strings shown when the SZS is loaded, they're not used in the patching process \
`FnameIdentifier` and `FnameNotIdentifier` are arrays of strings used to identify the loaded SZS, your patch gets selected only if the SZS contains ALL the files in `FnameIdentifier` 
and NONE of the files in `FnameNotIdentifier`. `FnameIdentifier` implicitly includes `MainLayoutName` and `SecondaryLayouts` \
`MainLayoutName` is the name of the layout file that renders the background. \
`MaintextureName` is the name of the texture that will be replaced. \
`PatchIdentifier` is the name of the pane that will be added, it is used to identify the patch so it should be unique for each patch. It must be shorter than 24 chars \
`targetPanels` is an array of strings of all the panes that should be removed from the main layout (so panes that already render the background), you need at least one because it's also used to detect where to place the new background pane. \
`SecondaryLayouts` contains the names of the layouts that use the texture that was replaced, you can find them from the advanced tab of the program.
`SecondaryTexReplace` is the name of the texture that will replace `MaintextureName`
# The Layout patches
Layout patches are slightly more complex and have nested properties but you can create them automatically :\
## Generating a layout patch with "LayoutDiff"
You need 2 SZS files : the one you patched and the original one, they must be for the same firmware version, the background image doesn't matter.
1) Open the Switch theme injector **Windows app** and open the PATCHED SZS
2) Go to the advanced tab and click on the "Layout Diff" button, it will show a dialog to open another SZS, open the ORIGINAL one.
3) If everything went ok you should be able to save your json patch.
4) Open the json you just saved with a text editor and change the name and the author
5) Look for panes you didn't actually change and remove them. (they're usually usd1 sections mistaken for panes, they have a weird name or no properties)
6) Save and share your layout
## The layout patch structure
A Layout patch starts with the `PatchName` and `AuthorName` which are self-explainatory, then there is a `Files` array which is an array of the following structure:
### LayoutFilePatch
This structure has 2 properties :
- `FileName` is the name of the target file in the SZS, path included. 
- `Patches` is an array of the changes that need to be applied, one per pane, it follows this structure : 
### PanePatch
This structure has only one compulsory property that is `PaneName`, the name of the pane to edit.
Then these are the properties that can be changed: (include only the ones you need to change) 
- `Position` : has 3 values X,Y and Z
- `Rotation` : has 3 values X,Y and Z
- `Scale` : has 2 values X and Y
- `Size` : has 2 values X and Y
- `Visible` : true or false

Only for picture panes (pic1) :
- `ColorTL` : Top left vertex color 
- `ColorTR` : Top right vertex color 
- `ColorBL` : Bottom left vertex color 
- `ColorBR` : Bottom right vertex color 
these values must be an hexadecimal color string, for example "FFFFFF" is white
