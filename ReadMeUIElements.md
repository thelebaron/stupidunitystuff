# stupid ui elements notes

must have ```EventSystem(UI Toolkit)``` in scene to process input actions

# show / hide
```
visualElement.style.display = DisplayStyle.None;
visualElement.style.display = DisplayStyle.Flex;
```

# buttons
register callbacks
```
button.RegisterCallback<ClickEvent>(ev => MyMethod()); //MouseUpEvent?
closeButton.clickable.clicked += MyMethod;
```
