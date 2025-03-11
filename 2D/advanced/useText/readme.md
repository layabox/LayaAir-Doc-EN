## Advanced Use of Text



LayaAir engine and IDE provide rich text components, suitable for various text application scenarios. This article introduces the display text components, bitmap text components, and input text components in the IDE respectively, so that everyone can have a comprehensive understanding of each text component of the LayaAir engine.

### 1. Display text component

The displayed text is the basic text that can only be viewed and cannot be edited or modified. LayaAirIDE provides two basic display text components, Text and Label.

#### 1.1. The underlying rendering mechanism of text

The LayaAir engine has two methods for underlying rendering of text.

They are the unified text submission method of the Text class, and the fillText single character drawing submission method of the Graphics class.

If Text text is used, the text characters in each Text text object will form a small picture collection and submit it to the large picture collection. For example, a text object in the game has 300 words, and if one of the characters changes, the corresponding The 300-word text atlas of the text object is re-introduced in the large image collection.

Or suppose, in a plot game, two people are having a conversation. One of the text objects needs to be continuously re-valued and assigned, causing the display content of the text object to change frequently. At this time, the Text object will continuously submit the text atlas to the large image collection. After the large atlas is filled, a new atlas needs to be created to continue submitting, instead of using the old atlas. If it does not reach the GC conditions will always exist, so if you check the memory usage statistics this time, you can see that the memory will continue to increase until the GC conditions are reached, and then it will be destroyed. And when the GC clears the garbage memory, it may cause instant lag.

When using fillText text drawing of the Graphics class, single characters are submitted to the large image collection, and characters that already exist in the large image collection will not be submitted repeatedly. Therefore, the utilization rate of the atlas is relatively good. Even when the text changes, it will not create a lot of junk atlases like Text text.

From this point of view, is fillText the best to use?

If the text does not need to be changed, then submitting the atlas by word is not as efficient as submitting it directly at once.

Therefore, we still have to look at the type of text, corresponding to the type of text used, and select the corresponding engine API, which will be better.

#### 1.2. Static text and dynamic text

Based on the mechanism of text rendering and submission in the previous section, and the difference between text used only for display or for interaction, we divide it into two concepts. Text that is only used for display and the text content does not need to be changed is called static text. Text that needs to change its content is called dynamic text.

Text and Label both belong to classes that handle static text. They are rendering mechanisms that uniformly submit every change to the text to a large atlas.

fillText, fillBorderText, strokeText, etc. are all text drawing methods used for single-character submission in the Graphics class.

In LayaAir IDE, Text and Label are static text components. However, text drawing methods such as fillText, fillBorderText, and strokeText of the Graphics class in the LayaAir engine are not encapsulated as components in the IDE, and the engine API needs to be used directly in the project code.

#### 1.3. Differences and choices in using different text APIs

If we have understood what static text and dynamic text are, then the choice of use is relatively simple. For example, there is no need to change some navigation and label buttons on the UI, and static text can be used. Text plots and tasks require dynamic text.

There are two types of static text components: Text and Label. How to choose?

Among them, Text is the most basic text component. It inherits directly from Sprite, while Label inherits from UIComponent and then Sprite. In principle, the shorter the path, the better the performance. Therefore, for relatively simple text, when Text can satisfy the requirements, Text should be used first in principle. Label, because of its rich attributes, is suitable for relative layout and other application scenarios that cannot be satisfied by Text.

For fillText, we not only need to understand its advantages, but also its shortcomings. Since the fillText text is drawn by the engine itself, it is currently only applicable to mainstream national languages ​​such as Chinese and English. For complex languages ​​such as Thai and Arabic, the display may be abnormal. Therefore, for these international language games with complex typesetting, the only way is to use system characters or bitmap characters of static text components to minimize the need for dynamic text modifications.

#### 1.4. Things to note when using

For static text, the Text component is preferred, because the performance is best when only static text is displayed. For dynamic text, fillText is preferred, unless requirements such as internationalization cannot be met.

For static text, when there are complex requirements such as relative layout, the Label component is more powerful.

The code usage of fillText is explained as follows:

```typescript
 /**
 	* Draw text on the canvas.
 	* @param text The text output on the canvas.
 	* @param x The x coordinate position (relative to the canvas) at which to start drawing text.
 	* @param y The y coordinate position (relative to the canvas) at which to start drawing text.
 	* @param font defines the font size and font, such as "20px Arial".
 	* @param color defines the text color, such as "#ff0000".
 	* @param textAlign text alignment, optional values: "left", "center", "right".
 	*/
	fillText(text: string | WordText, x: number, y: number, font: string, color: string, textAlign: string): FillTextCmd {
    	return this.addCmd(FillTextCmd.create(text, x, y, font, color, textAlign, 0, ""));
	}
```

Here we focus on the difference between the font attribute in fillText and several other text font attributes. The sample code for fillText is as follows:

```typescript
    /**
 	* Executed after the component is activated. At this time, all nodes and components have been created. This method is only executed once.
 	*/
	onAwake(): void {
    	let sprite = new Laya.Sprite();
    	Laya.stage.addChild(sprite);
    	sprite.graphics.fillText("fillText: text drawn by the engine, suitable for simple single-line text", 100, 300, " 60px simHei ", "#ff0000", "left");
	}
```

In Text and Label, font only refers to the system font. For example, whether the text is `Microsoft YaHei` or `Arial`, just select or enter the corresponding font. In fillText, font is the setting value of font and font size. All related text styles (size, font, italics, bold) can also be specified here.

For example, when the value is `60px simHei`, the text size is set to 60 pixels, using the simHei font. As shown in Figure 1-1 below:

![1-1](img/1-1.png)

(Picture 1-1)

A value of `60px simHei italic` adds an italic effect. As shown in Figure 1-2 below:

![1-2](img/1-2.png)

(Figure 1-2)

The value `bold 60px simHei italic` adds text bolding effect. As shown in Figure 1-3 below:

![1-3](img/1-3.png)

(Figure 1-3)

> Tips:
>
> - Size and font must be set, and the font must be placed behind the px size of the text.
> - bold needs to be placed in front of the text px size.

### 2. Bitmap display text component

Bitmap display text is a kind of text based on bitmap. Although it displays the content of the text, its essence is still a bitmap. Usually used for fonts that are not common on the system or personalized fonts for art design. The compatibility and beauty of the fonts can be ensured through bitmaps. For games with strict text effect and position requirements, it is recommended to use bitmap text.

> Ordinary text may have pixel-level position offsets under different operating environments. For example, there are also positional offsets between low and high versions of Chrome. Therefore, for static UI texts with high requirements, bitmaps can be used to display text.
>
> Bitmap text is suitable for text that does not require a large number of words. It is often used for title text such as level X and combo X. It is not suitable for games with text plots.

#### 1. FontClip font clipping component

The FontClip component is used by LayaAir to divide the complete bitmap into independent text units based on the principle of equal cutting, and then arbitrarily clip and splice them together. The effect is shown in Figure 2-1.

![2-1](img/2-1.png)

(Figure 2-1)

The numbers 0-9 in Figure 2-1 are the display of the original art image effect, and the number 999 is the effect of the FontClip component.

In the property panel on the right side of Figure 2-1, the sheet value corresponds to the 9 numbers of the bitmap. At this time, the numbers must be completely corresponding, because this value is not only the reference basis for the engine to cut the number of copies, but also the text mapping relationship. Only by matching, whatever value is entered in the attribute value will be displayed, such as 999 in Figure 2-1. And the FontClip component can also control effects such as the bitmap text spacing after cutting.

If the bitmap text is not numbers, but letters or Chinese, etc., and the content is relatively large, the text can also be arranged in multiple lines when the art is drawn, but each text must be arranged with equal height, equal width and equal spacing. At this time, the sheet value used for mapping needs to add a space at the end of each line to let the engine know that the bitmap text is arranged in multiple lines. Results as shown below:

![2-2](img/2-2.png)

(Figure 2-2)

From Figure 2-2, we can see that the sheet attribute value `Rat, Ox, Tiger, Rabbit, Dragon, Snake, Horse, Sheep, Monkey, Rooster, Dog and Pig Year` corresponds to the text on the bitmap, and a space is added after the sheep. At this time, you can The value input text characters on the bitmap, and we also show the effect of vertical typesetting, indicating that the cut text can also be controlled by typesetting.

Careful developers will find that the bitmap characters are in traditional Chinese and the sheet values ​​are in simplified Chinese, but the display effect is not affected. This is because the sheet value only serves as a mapping relationship. Even if the text of the twelve zodiac signs corresponds to the zodiac picture, the corresponding zodiac picture will be displayed. However, the number on the mapped text and bitmap must be complete, otherwise the wrong number of slices will lead to display errors after the bitmap is clipped.

Finally, let’s talk about spaces. If the current FontClip component is oriented to a single line of text as shown in Figure 2-1, the sheet value does not support spaces. For the multi-line text in Figure 2-2, the space in the sheet value represents a line break, not a blank space. If you add a space to the Value value, you can see from Figure 2-2 that a complete text position will be vacant. In fact, it is not just spaces. All text that does not exist in the sheet value will be filled with empty complete characters when entered in the value.



#### 2. BitmapFont bitmap font

Usually FontClip can already meet the needs of single-line bitmap text such as levels and special effects. If there are special circumstances where you want spaces or artistic fonts that are not available in some systems, you can also use BitmapFont bitmap fonts to achieve this. In LayaAir IDE, you can directly create and produce bitmap fonts. This function will be introduced below.

As shown in Figure 2-3, in the project resource panel, right-click to create a `bitmap font`.

![2-3](img/2-3.png)

(Figure 2-3)

After creation, as shown in Figure 2-4, a `BitmapFont.fnt` font file will be generated.

![2-4](img/2-4.png)

(Figure 2-4)

After clicking to select, you can create a bitmap font in the property settings panel as shown in Figure 2-5.

![2-5](img/2-5.png)

(Figure 2-5)

Click the `+` of "Edit Character Set" to create a character, as shown in Figure 2-6:

![2-6](img/2-6.png)

(Figure 2-6)

| <span style="display:inline-block;width:80px">Parameters</span> | Meaning	|
| --------------------------------------------------------- | ------------------------------------------------------------ |
| Allow scaling	| When checked, the text can be scaled and displayed according to the font size setting in the text. If this is not checked, the font size setting will be ignored and only the actual size of the bitmap text will be displayed |
| Font size	| After checking Allow scaling, the font size here is used for scaling adjustment of the actual text font size. For example, if the font size of the actual text is set to 24 and the font size of the bitmap font is set to 12, then the bitmap font will eventually be enlarged by 1 times for display. It is recommended here to be consistent with the "font size" attribute of the text component, so that the obtained The zoom effect is the most accurate. If scaling is not allowed, the font size setting here and the actual text will have no effect, and the source image size of the bitmap font will always be maintained |
| Custom row height	| After checking, you can set the row height. If unchecked, the default row height is used. Line height determines the height of each line when text is displayed in multiple lines, and is mainly used for typesetting text in the vertical direction. If it is 0, the font size is used as the line height. If greater than 0, this is the actual text line spacing (the distance between the upper boundaries of two bitmap fonts). After creation, you can set the line height in the "leading" attribute of the text component |
| character	| The character corresponding to the picture only supports single characters, which can be numbers or strings.	|
| Pictures	| A picture in the resource library	|
| Placeholder	| If the value is 0, it represents the horizontal occupancy width of a character after creation, which is determined by the width of the character image. If the value is greater than 0, use that value as the kerning between characters (the distance between the left borders of two bitmap fonts) |

As shown in Figure 2-7, there are twelve pictures in the resources folder, which are bitmaps of the twelve characters "rat, cow, tiger, rabbit, dragon, snake, horse, sheep, monkey, chicken, dog, pig". These twelve pictures are used as bitmaps. Figure font. The height of the image is 151 px, so the line height is set to 151. The placeholder defaults to 0, and after creation it will become 144, which is the width of the image. After editing, click "Apply". After clicking, another BitmapFont.png image will be created for previewing the bitmap font.

![2-7](img/2-7.png)

(Figure 2-7)

Bitmap fonts can be selected in the Font property of text components such as Text, Label, TextInput, and TextArea, and the font size and line spacing can be adjusted. The following takes Text as an example to demonstrate the use of bitmap fonts. As shown in the animation 2-8, select the bitmap font you just created in Text, and then enter the "character" corresponding to the bitmap font in the text to use the bitmap font.

![2-8](img/2-8.gif)

(Animation 2-8)

> Tips：
>
> Developers can also use external tools to complete the production and then import it for use. Here is an open source tool recommended: https://snowb.org/, which can be produced online.

### 3. HTML, UBB, template

HTML tags are used on all text-related UI components, and can be mixed with other UBB tags, text templates, and ordinary text. Developers only need to set the corresponding syntax attributes. Please refer to the documentation for supported syntax and usage methods. ["Basic Text"](../../displayObject/Text/readme.md).

### 4. Input text component

The input text components are the single-line input component TextInput and the multi-line text area component TextArea.

You can actually tell the difference from the component names.

One is that it cannot wrap lines and is used for single-line input. For example, those single-line input boxes on registration information use TextInput.

The other is a multi-line text component TextArea that can wrap lines, such as personal introduction, notes, etc. And the TextArea text component supports a vertical scroll bar on the right side and a horizontal scroll bar on the bottom.

Because the usage of the input text component is relatively simple, the component is also relatively easy to understand. I won’t introduce it in detail here. During use, you can try it by following the prompts or directly read the corresponding component attribute description document.

### 5. Performance optimization of text

### 5.1 Use corresponding text components as much as possible

Each different component has its own unique function, so you must fully understand the function of the component and use it according to its characteristics.

As mentioned above, the performance of the Text component for static display of text is the best. Therefore, during project development, Text components should be used as much as possible. Use Label when the Text component is not satisfied.

For static text that does not need to change the content, when the text component has many nodes, we can turn on cacheAs, set normal to optimize the number of nodes, and set bitmap to optimize the number of DrawCalls.

When the text content needs to be changed, even if only one character is changed, if FillText can meet the needs, then the FillText text component should be used first as much as possible. When FillText cannot satisfy the requirement, secondly consider whether the bitmap text component can satisfy the requirement, and finally consider the static text component.

### 5.2 Avoid using text strokes whenever possible

At runtime, text with a stroke calls the drawing command one more time than text without a stroke. At this time, the CPU usage of text is proportional to the number of texts. Therefore, try to use alternatives to accomplish the same needs. For example, when the amount of characters is small, bitmap text can be used.

### 5.3 Optimization of DrawCall interrupted by text

In 2D UI layout, if developers mix and arrange text with other UI nodes, it will inevitably interrupt the merged rendering of the atlas, increase the number of DrawCalls, and generate unnecessary performance overhead. So, there are two options to optimize.

First, when editing the layout in LayaAir IDE, arrange all text components sequentially in the nodes and do not intersperse them with other UI components.

Second, there is a drawCallOptimize attribute in the component in LayaAir IDE. We set the true value for the drawCallOptimize attribute of the parent node of all texts. When this is set, the engine will automatically enable text merging optimization and extract all text to the top layer. Developers no longer need to deliberately adjust the sorting of atlas resources and texts to achieve the goal of automatic optimization of drawCall, and the optimization will be more Just be thorough.

> It should be reminded that the drawCallOptimize optimization solution will automatically raise the text to a display level, so it is not suitable for special needs where the text must be semi-occluded. Of course, in most cases the text should be displayed in full. If it is fully hidden, you can directly set the hidden attribute. Therefore, it is recommended that developers enable this optimization plan.













