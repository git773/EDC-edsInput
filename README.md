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
`npm install --save-dev @types/react-dom`  
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

If you follow my lead, you can do this with two properties: One for value, one for label.  
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

And secondly, there is this: `of-type="SingleLine.Text" usage="bound" required="true"`. `required` attribute is clear enough, but `usage` and `of-type` could use some explaining. `usage` can be one of two things: **bound** or **input**. These values refer to the flow of data. If it is input, that means that the component will just receive data from Dataverse / user and be happy with it. This makes sense for the label property. Bound on the other hand, means that the data will be able to flow both ways, and that the component will be able to influence the value of the Dataverse column it is bound to on setup. Interesting.

`of-type` sets the expected format of the property value. The list of formats can be found in the reference. It is also possible to define something similar to custom format by defining the `type-group` and setting the `of-type-group` attribute. For Input label and value, `SingleLine.Text` is the best choice.

The properties are done, the only thing left are resources and APIs. Resources are important because they connect your component manifest to the component implementation. You can add additional resources if you need them, but at minimum you should always have **index.ts** in there. For the APIs, you won't be needing any of those for this simple Input component, so it is ok to delete the extra green and admire your clean and finished Manifest.

<p align = "center">
<img src = "https://user-images.githubusercontent.com/75603877/139607744-0f1954b9-1817-4876-aa7a-ad5a8aa573e2.png"><br>
One down, two to go. All this text for 13 lines of code.
</p>

Good work so far! Now that the Manifest is in place, run `npm run build` to get the other manifest (**ManifestTypes.d.ts**) generated according to your definition.


### 3. Implement the React component

Let's make another file in your busy folder for you to worry about. This is going to be your react component. So make the file **edsInputWrap.tsx** and add all the boilerplate for a react class.

```
import * as React from 'react';
import { Input, Label } from '@equinor/eds-core-react';

export interface edsInputWrapProps {}

export interface edsInputWrapState extends React.ComponentState, edsInputWrapProps{}

export class edsInputWrap extends React.Component<edsInputWrapProps,edsInputWrapState> {

    constructor(props: edsInputWrapProps) {
        super(props);
        this.state = {};
    }

    render():JSX.Element{
        return (<div>
            
        </div>);
        
    }
}
```

It is important to add both the properties and the state. The properties are the interface towards the PCF while the state is important to make sure the component is responsive. For the properties, you need the value, label and a way to signal something has changed.  

```
export interface edsInputWrapProps {
  value?:string;
  label?:string;
  valueChanged?:(newValue:string)=>void;
}
```  

The value is the most important for the state, so in the constructor, pass the state the initial value.  
`this.state = {value: props.value || undefined}`  

Implement the render function.  

```
render():JSX.Element{
    const {value} = this.state;
    return (
        <div>
          <link rel="stylesheet" href="https://eds-static.equinor.com/font/equinor-font.css"/>
          <Label htmlFor="textfield-normal" label={this.props.label||""} />
          <Input
            id="textfield-normal"
            value={value}
            onChange={this.onChangeEvent}
           />
        </div>
    );
}
```  

There are several important lines there. First, it is important to pick up the value from the state, and not the properties. State is going to change and we want that reflected on the screen. Second, fixed label is fine. If there is no label passed down, it is probably a mistake, since it is required in the component, but in that case we display the empty string. And finally, the onChange event handler is necessary to do the updates to the component on change. (both React component and passing the data back to the PCF component require an onChange handler)  

So, the next thing to add is the onChange handler: onChangeEvent.  

```
private onChangeEvent = (event: React.FormEvent<HTMLInputElement>): void => {

    this.setState(
        (prevState:edsInputWrapState):edsInputWrapState => {
            prevState.value = event.currentTarget.value;
            return prevState;
        }
    );

    if(this.props.valueChanged) {
        this.props.valueChanged(event.currentTarget.value);
    }

}
```  
And just as before, the first part `setState()` is concerned about updating the internal state of the react component, while the second part is there to signal to the PCF component that something has changed. Once you get to testing part, try removing one statement or the other, and see what happens! It is kind of silly!  

And if you got this far, then you have finished the implementing the react component! Well done! Now just one more thing left.

<p align = "center">
<img src = "https://user-images.githubusercontent.com/75603877/139726682-bb57261f-a238-421b-9833-a601a5339abd.png"><br>
I am not a React expert so this might not be best practice. If you can better, I will be happy to learn from you. :)
</p>  

### 4. That index.ts really tied the component together  

This is going to be cool! Index.ts is the implementation of the PCF component, and it is the link between the Manifest and the React component. The template comes with a lot of stuff inside, but most are comments, and the rest are standard functions. So, this will be a breeze.  

First step, import what needs importing.
```
import * as React from 'react';
import * as ReactDOM from 'react-dom';
import { edsInputWrap, edsInputWrapProps } from './edsInputWrap';
```  
That proud moment when you import your component; like a pro!

Ok, cool, so what do you actually need? Well, definitely not a constructor, you can ignore that one. But you need some private fields, like `container` for the HTML Div Element that is going to hold you component, `notifyOutputChanged` function to signal to the component that it needs to update, and props that you can use to pass the data around.  
```
private container: HTMLDivElement;
private notifyOutputChanged: () => void;
private props: edsInputWrapProps = {valueChanged: this.valueChanged.bind(this)};
```  
It is important to notice that the valueChanged event handler from the properties was connected to another valueChanged function. That one needs to be implement here in **index.ts**. And its purpose will be to update the props value when it changes and update the output/view based on it. Here is how it can look like:

```
private valueChanged(newValue: string) {
    if (this.props.value != newValue) {
        this.props.value = newValue;
        this.notifyOutputChanged();
    }
}
```

With the complicated part is out of the way, let's get back to standard functions. **Index.ts** should consist of `constructor()`, `init()`, `updateView()`, `getOutputs()`, `destroy()`. `init()` is there to start off the component; an entry function. It captures the data from the context and it is convenient to store its arguments internaly in the component fields for use in other functions.  

```
this.container = container;
this.props.value = context.parameters.value.raw || undefined;
this.props.label = context.parameters.label.raw || undefined;
this.notifyOutputChanged = notifyOutputChanged;

```

`getOutputs()` is there to capture the changes from the **bound** properties (those that can serve as both inputs and outputs, check the **ManifestTypes.d.ts**) and pass their values back to Dataverse. In this case, only bound property is value, so that is what you should use.  

```
return {value: this.props.value};
```  

`destroy()` is just a cleanup function to manage memory and get rid of the container.  

```
ReactDOM.unmountComponentAtNode(this.container);
```

And the most interesting of all is `updateView()` which interfaces directly with the React component and renders it when the `notifyOutputChanged()` is called. First you should get fresh value and label from the context, and then pass those on to your React component.

```
this.props.value = context.parameters.value.raw || undefined;
this.props.label = context.parameters.label.raw || undefined;
ReactDOM.render(
    React.createElement(
        edsInputWrap,
        this.props
    ),
    this.container
);
```  

This is the result, cleaned from the extra comments.

<p align = "center">
<img src = "https://user-images.githubusercontent.com/75603877/139726694-05fbf672-1a5d-4efd-bc6e-ce64842bcc28.png"><br>
The last chunk of code.
</p> 
