# Option Box

> Author: Charley

In the `LayaAir` engine's classic `UI` system, option box functionality has four different application scenario components: Dropdown Option Box `ComboBox`, Radio Button `Radio`, Radio Button Group `RadioGroup`, and CheckBox `CheckBox`.

### 1. Dropdown Option Box (`ComboBox`)

The dropdown option box lists all available options through a dropdown list. Users can select one option from them as the display result and trigger related logic.

Commonly used for menu selection, difficulty selection, setting options, and other scenarios that require selecting one from multiple options.

**For details, see ["Dropdown Option Box Component"](../ComboBox/readme.md)》**

### 2. Radio Button (`Radio`)

The radio button is a state toggle button component inherited from the button component, referred to as **Radio**.

Radios are usually not used independently but are used in combination with `RadioGroup`.

**Radio** cannot toggle selected state through repeated clicking like **CheckBox**.

Its selected state can only be changed through **mouse selection** (remains selected after selection) or **through code modification**.

**For details, see ["Radio Button Component"](../Radio/readme.md)》**

### 3. Radio Button Group Container (`RadioGroup`)

The radio button group is a container component used to hold multiple radio buttons, implementing mutually exclusive selection through internal logic.

In the same **radio button group container**, users can only select one radio button at a time. Others will automatically be deselected. This exclusive radio selection logic achieves a **unique selection** effect without manually writing control code.

**For details, see ["Radio Button Group Container Component"](../RadioGroup/readme.md)》**

### 4. CheckBox Button (`CheckBox`)

The checkbox button is also inherited from the button component. It's a state button that can repeatedly toggle selected state, referred to as **CheckBox**.

Unlike radios which almost never exist independently, CheckBoxes can exist independently or be used in combination.

Users can achieve **free toggling between selected and unselected states** through repeated clicking.

CheckBoxes are suitable for scenarios requiring **multiple selection** or **free toggling**, such as game settings, option confirmation, etc.

Since they're often used for multiple selection operations, they're also called **multiple selection button**.

**For details, see ["CheckBox Component"](../CheckBox/readme.md)》**
