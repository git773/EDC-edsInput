# EDC-edsInput
Lab on how to make a Power App Component Framework component wrap for the Equinor Design System React Input component.

Hello! Let's roll:

### 0. Make sure you have everything installed:

1. [Visual Studio Code](https://code.visualstudio.com/) 
2. [Node.js](https://nodejs.org/en/)
3. [Microsoft Power Platform CLI](https://docs.microsoft.com/en-us/powerapps/developer/data-platform/powerapps-cli#install-power-apps-cli)
4. [Build Tools for Visual Studio](https://visualstudio.microsoft.com/) or .NET build tools (msbuild is what you need)

Or just use [codespaces](https://github.com/features/codespaces)! You can start from [here](https://github.com/Equinor-Playground/powerapps-codespaces) :)

### 1. Set everything up

Start by making a new folder for your component and open it in VSCode. Open a terminal in VSCode and to create a PCF project:  
`pac pcf init --namespace <namespace> --name <name> --template field`  

For example: `pac pcf init --namespace edcPCF --name edsInput --template field`  
*field means the component will be primarily used on a form linked to a field. That makes sense for an input component, but if you are building a grid/view then you should choose `--template dataset`.*

Since EDS is using react components, you also need to add the following:  
`npm install --save-dev @types/react`  
`npm install --save-dev @types/reactdom`  
`npm install @equinor/eds-core-react styled-components`  
`npm install typescript --save-dev`

Your folder is looking busy? Cool, then you're good to go!  

<p align = "center">
<img src = "https://user-images.githubusercontent.com/75603877/139604371-ac3374f7-d195-44e2-9998-1ed767857a07.png"><br>
Busy folder looking busy.
</p>


### 2. Fix the Manifest

There are only two files in the busy folder you should worry about for now. One is **index.ts** and another is **ControlManifest.Input.xml**. Let's start with the latter.  

Open **ControlManifest.Input.xml** and look at control tag on **line 3**:  
`<control namespace="edcPCF" constructor="edsInput" version="0.0.1" display-name-key="edsInput" description-key="edsInput description" control-type="standard" >`  
The `display-name-key` and `description-key` you put in here will be visible to people using the component so make sure to choose a good name. Think of the component as your baby, you would not want other components making fun of its name! 

Here is an example of a very badly named componet:  
`<control namespace="edsPCF" constructor="edsInput" version="0.0.1" display-name-key="Field" description-key="Field to format according to EDS Text Input" control-type="standard" >`  
<p align = "center">
<img src = "https://user-images.githubusercontent.com/75603877/139605014-6184bb5e-e644-4d0e-bbb4-dc4bdca5d76c.png"><br>
The unfortunate control called Field.
</p>

Just for reference, I will keep mine named edsInput. It has a nice ring to it.

Next, you will encounter a lot of green commented stuff. Feel free to remain unconcerned. Skip or delete, whichever you prefer. I will just clear everything from **line 4** to **line 19**. Because the properties are the thing you should focus on.  

Properties are the interfaces between the user, Dataverse and your component. How to decide what you need? Easy, just follow the EDS - Equinor Design System! Also, this is a good place to check out the reference in the docs so I will just leave the link here for you to explore: [reference](https://docs.microsoft.com/en-us/powerapps/developer/component-framework/manifest-schema-reference/property).  

Looking at the [EDS Storybook](https://eds-storybook-react.azurewebsites.net/?path=/docs/components-input--default) you can decide how complicated do you want your life to be. For my sake, I am keeping it simple. I will make it variant default, type text. I want a label though, because it looks nice. No placeholder necessary. And no strange states like read only and disabled. Just an nice, labeled, default, text Input. :)

If you follow my lead, we can do this with two properties: One for value, one for label.  
```
<property name="value" display-name-key="Value" description-key="Single line text input" of-type="SingleLine.Text" usage="bound" required="true" />
<property name="label" display-name-key="Label" description-key="Label of the text input" of-type="SingleLine.Text" usage="input" required="true" />
```

There are a couple of things to unpack here. First, the name and description are again important. Remember the unfortunate component? Well here is its friend, the unfortunate property.

`<property name="edsPCF" display-name-key="Property_Display_Key" description-key="Property_Desc_Key" of-type="SingleLine.Text" usage="bound" required="true" />`
<p align = "center">
<img src = "https://user-images.githubusercontent.com/75603877/139606263-253e1c06-b82d-4754-9348-63cfdfe614ff.png"><br>
Can you guess what this property does? Neither can an unsuspected citizen developer.
</p>

And secondly, we have `of-type="SingleLine.Text" usage="bound" required="true"`. `required` attribute is clear enough, but `usage` and `of-type` could use some explaining. `usage` can be one of two things: **bound** or **input**. These values refer to the flow of data. If it is input, that means that the component will just receive data from Dataverse / user and be happy with it. This makes sense for the label property. Bound on the other hand, means that the data will be able to flow both ways, and that the component will be able to influence the value of the Dataverse column it is bound to on setup. Interesting.

`of-type` sets the expected format of the property value. The list of formats can be found in the reference. It is also possible to define something similar to custom format by defining the `type-group` and setting the `of-type-group` attribute. In case of the input field, for both the label and the value, `SingleLine.Text` is the best choice.

Now the properties are done, the only thing left are resources and APIs. Resources are important because they connect your component manifest to the component implementation. You can add additional resources if you need them, but at minimum you should always have **index.ts** in there. For the APIs, you won't be needing any of those for this simple Input component, so it is ok to delete the extra green and admire your clean and finished Manifest.

<p align = "center">
<img src = "https://user-images.githubusercontent.com/75603877/139607744-0f1954b9-1817-4876-aa7a-ad5a8aa573e2.png"><br>
One down, two to go. All this text for 13 lines of code.
</p>


