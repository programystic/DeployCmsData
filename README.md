# Deploy CMS Data

Deploy CMS Data is a component that you can use for your Umbraco application which allows you to build migrations that deploy CMS updates.

The updates are run on startup, and the status is saved to the database so that once they are run successfully, they are never run again.

---

Version|Umbraco Version|Status
--- | --- | ---
7.6.0.3 | 7.6.0|Published to Nuget
8.0.0 | 8.0.0|Work in progress
7.13.0 | 7.13.0|Work in progress
7.4.0 | 7.4.0|Work in progress
7.0.0 | 7.0.0|Work in progress

---

## Creating an Upgrade script

### Install from Nuget

```PM> Install-Package DeployCmsData.UmbracoCms -Version 7.6.0.3```

Create a new class that implements IUpgradeScript

```csharp
using DeployCmsData.Core.Interfaces;

public class Script01 : IUpgradeScript
{
    public bool RunScript(IUpgradeLogRepository upgradeLog)
    {
        return true;
    }
}
```
## Naming of upgrade scripts
The scripts are run on startup in name order. So you will need to make sure you name them correctly so that they are run in the correct order.

The easiest way is to use the same name with a numeric value postfix. 

Example: ```Script0010.cs, Script00020.cs, Script00010.cs ...```

---

### RunScriptEveryTime Attributes

The script is only ever run once, so during development you might want to add the following attribute so that the script is run every time on startup, even if it's already been run.

```csharp
[RunScriptEveryTime]
public class Script01 : IUpgradeScript
```

--- 

## DeployCmsData Builders

Within our upgrade script we can use builders to manipulate the CMS data.

## Document Type Folder Builder

To create new folders to structure your document types:
```csharp
new DocumentTypeFolderBuilder("Compositions")
    .BuildAtRoot();

new DocumentTypeFolderBuilder("Composition Sub-Section")
    .BuildWithParentFolder("Compositions");
```

## Document Type Builder

The basic command to create a new document type is:
```csharp
new DocumentTypeBuilder("contentBase")
    .BuildAtRoot();
```

To specify a name, icon and folder:
```csharp
new DocumentTypeBuilder("pageMetaData")
                .Name("Page Meta Data")
                .Icon(Icons.MindMap)
                .BuildInFolder("Compositions");
```

To add fields to the document type:
```csharp
 var builder = new DocumentTypeBuilder("contentBase");

builder
    .Name("Content Base")
    .Icon(Icons.Document);

builder.AddField("pageTitle")
    .Description("The title of the page, this is also the first text in a google search result. The ideal length is between 40 and 60 characters")
    .IsMandatory()
    .Tab("Content");

builder.AddField("bodyText")
    .Name("Content")
    .DataType(DataType.RichTextEditor)
    .Tab("Content");

builder.BuildInFolder("Compositions");
```

To add existing document types as compositions to the document type:
```csharp
var builder = new DocumentTypeBuilder("homePage");

builder
    .Name("Home Page")
    .Icon(Icons.Home)
    .AddComposition("pageMetaData")
    .AddComposition("contentBase")
    .AddComposition("navigationBase");

builder.AddField("mainContent")
    .DataType(LocalDataTypes.Grid)
    .Tab("Content");

builder.BuildInFolder("Pages");
```

---

## MultiNode TreePicker Builder
To create a new MultiNode TreePicker data type:

```csharp
var myDataTypeid = Guid.Parse("{60FFCA99-3B98-49EA-9F64-E4C69BB00285}");

new MultiNodeTreePickerBuilder(LocalDataTypes.MultiNodeTreePicker)
    .Name("Multi Node Tree Picker")
    .ShowOpenButton()
    .Build();
```
And with more options:
```csharp
var myDataTypeid = Guid.Parse("{60FFCA99-3B98-49EA-9F64-E4C69BB00285}");

new MultiNodeTreePickerBuilder(myDataTypeid)
    .Name("Another Multi Node Tree Picker")
    .AllowItemsOfType("type1", "type2")
    .MinimumNumberOfItems(1)
    .MaximumNumberOfItems(5)
    .ShowOpenButton()
    .Build();
```
---
## Grid DataType Builder
To create a new grid data type:
```csharp
var myDataTypeid = Guid.Parse("{3D7B34BF-1E2E-4D49-A040-1C25D14E2074}");

new GridDataTypeBuilder(myDataTypeid)
    .Name("Another Grid View")
    .AddStandardToolBar()
    .AddStandardRows()
    .AddLayout("1 column layout", 12)
    .AddLayout("2 column layout", 4, 8)
    .AddLayout("3 column layout", 4, 4, 4)
    .AddLayout("4 column layout", 3, 3, 3, 3)
    .Build();
```
```csharp
new GridDataTypeBuilder(LocalDataTypes.Grid)
    .Name("Default Grid View")
    .Columns(12)
    .AddLayout("1 column layout", 12)
    .AddLayout("2 column layout", 4, 8)
    .AddLayout("3 column layout", 4, 4, 4)
    .AddLayout("4 column layout", 3, 3, 3, 3)
    .AddRow("1 column", 12)
    .AddRow("2 columns", 6, 6)
    .AddRow("2 columns (left)", 4, 8)
    .AddRow("2 columns (right)", 8, 4)
    .AddRow("3 columns", 4, 4, 4)
    .AddRow("4 columns", 3, 3, 3, 3)
    .AddStandardToolBar()
    .Build();
```
---

## Template Builder

To add existing views:

```csharp
new TemplateBuilder("Master") // The alias must match the view name (master.cshtml)
    .Build();
```
And to specify a master template:
```csharp
var homeTemplate = new TemplateBuilder("Home")
    .WithMasterTemplate("Master")
    .Build();
```