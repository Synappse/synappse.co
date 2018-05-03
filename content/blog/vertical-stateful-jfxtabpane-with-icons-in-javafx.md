+++
author = "Sebastian Suchanowski"
categories = ["Java", "JavaFX", "Cross-Platform", "Tutorial"]
date = "2018-05-03"
description = "In this post, I will show you how to build a custom vertical tab bar with icons using JFXTabPane. Our main goal over here is to create a side menu with 1 tab being single state logout button. Let's take a look at the final result:"
featured = "vertical-stateful-jfxtabpane-with-icons-in-javafx.jpg"
featuredalt = "Vertical, stateful JFXTabPane with icons in JavaFX"
featuredpath = "img"
linktitle = ""
title = "Vertical, stateful JFXTabPane with icons in JavaFX"
type = "post"
+++

A while ago I've started working on a product that needed a multiplatform desktop app. I choose JavaFX as due to my research it will work the best with the other tools and frameworks that are necessary for this project.

## Task

In this post, I will show you how to build a custom vertical tab bar with icons using `JFXTabPane`. Our main goal over here is to create a side menu with 1 tab being single state logout button. Let's take a look at the final result:

<center>
    ![vertical tab pane final result](/img/vertical-tabpane-final-result.gif)
</center>

**User Profile** and **Settings** tabs are moving us to the respected views with different content. **Logout** is a custom implementation of the tab and it doesn't have any content to show - so it has to remember the last open tab and stay there while the action is being triggered.

## Step by step solution

### Prepare FXML file

The first thing we need to do is to add a `JFXTabPane`, then add single tabs (`Tab`), configure and style all of it.

In order to get the `JFXTabPane` class, we need to add [jfoenix](http://www.jfoenix.com) library to our project. I am using IntelliJ so I went to `File > Project Structure > Libraries > From Maven` and there I've searched for `jfoenix`. It should look like this:

<center>
    ![adding jfoenix library to project](/img/javafx-adding-jfoenix-library.jpg)
</center>

Then we need to open [**SceneBuilder**](http://gluonhq.com/products/scene-builder/) (you can do all of that from code) and prepare the fxml file adding there:

* `StackPane` - in case we need to show a dialog (I will not add the dialog in this tutorial but there is a perfect place to do it after the user clicks on the **Logout** button)
* `JFXTabPane` - which will hold all `Tab`s for us
* `Tab` x3 with `AnchorPane` inside them

<center>
    ![adding jfoenix library to project](/img/scene-builder-views-hierarchy.jpg)
</center>

After that choose `Side: LEFT` from `Properties` inspector or you can do it from code:

```
tabContainer.setSide(Side.LEFT);
```

### Configuration in Controller class

Let's get now into connecting everything. Explained better with a piece of code.

````
/// 1.
@FXML
private JFXTabPane tabContainer;

@FXML
private Tab userProfileTab;

@FXML
private Tab settingsTab;

@FXML
private Tab logoutTab;

/// 2.
private double tabWidth = 90.0;
public static int lastSelectedTabIndex = 0;

private void configureTabPane() {
    /// 3.
    tabContainer.setTabMinWidth(tabWidth);
    tabContainer.setTabMaxWidth(tabWidth);
    tabContainer.setTabMinHeight(tabWidth);
    tabContainer.setTabMaxHeight(tabWidth);
    tabContainer.setRotateGraphic(true);
    
    /// 4.
    configureTab(userProfileTab, "User\nProfile", "/co/synappse/project01/resources/images/user-profile.png");
    configureTab(settingsTab, "Settings", "/co/synappse/project01/resources/images/settings.png");
    configureTab(logoutTab, "Logout", "/co/synappse/project01/resources/images/logout.png");
}

private void configureTab(Tab tab, String title, String iconPath) {
    double imageWidth = 40.0;

    /// 5.
    ImageView imageView = new ImageView(new Image(iconPath));
    imageView.setFitHeight(imageWidth);
    imageView.setFitWidth(imageWidth);

    Label label = new Label(title);
    label.setMaxWidth(tabWidth - 20);
    label.setPadding(new Insets(5, 0, 0, 0));
    label.setStyle("-fx-text-fill: black; -fx-font-size: 8pt; -fx-font-weight: normal;");
    label.setTextAlignment(TextAlignment.CENTER);

    BorderPane tabPane = new BorderPane();
    tabPane.setRotate(90.0);
    tabPane.setMaxWidth(tabWidth);
    tabPane.setCenter(imageView);
    tabPane.setBottom(label);

    /// 6.
    tab.setText("");
    tab.setGraphic(tabPane);
}
````

Ad. 1 - All the references we need to our views for the time being.

Ad. 2 - We need to define a menu tile size as we will be building it up from scratch in code.

Ad. 3 - Setting mentioned size to our `JXFTabPane`.

Ad. 4 - Configure each tab with icon and title.

Ad. 5 - Setup `ImageView` with icon file and size, `Label` with a title and align everything within `BorderPane` which will be used as new `Tab` content.

Ad. 6 - Clear any text that might have been set in fxml and set the newly created content for the tab.

### Add style

Now we need to style our menu bar - we will use some of this code to manually set and unset selection for tabs.

```
.root {
    -fx-base: white;
    -fx-color: -fx-base;

    -fx-font-family: "Montserrat";
    -fx-accent: rgb(246, 246, 246);
    -fx-default-button: -fx-accent;
    -jfx-primary-color: -fx-accent;
    -jfx-light-primary-color: -fx-accent;
    -jfx-dark-primary-color: -fx-accent;
    -fx-focus-color: rgb(222, 222, 222);
    -jfx-secondary-color: -fx-focus-color;
    -jfx-light-secondary-color: -fx-focus-color;
    -jfx-dark-secondary-color: -fx-focus-color;
}

 /* JFX Tab Pane */
.jfx-tab-pane .headers-region {
    -fx-background-color: -fx-accent;
}

.jfx-tab-pane .tab-header-background {
    -fx-background-color: -fx-accent;
}

.jfx-tab-pane .tab-selected-line {
    -fx-stroke: -fx-accent;
}

.jfx-tab-pane .tab-header-area .jfx-rippler {
    -jfx-rippler-fill: -fx-focus-color;
}

.tab-selected-line {
    -fx-background-color: -fx-focus-color;
}
```

I've created a `global.css` file and then set the style to my root view in fxml.

<center>
    ![setting style to project javafx](/img/setting-style-to-view.jpg)
</center>

### Add support for single and double state actions

We are almost finished right now. Because we want to have a custom style of selected tab we need to add implementation to `setOnSelectionChanged`.

First of all inside `configureView()` we have to add an `EventHandler`.

```
/// 7.
EventHandler<Event> replaceBackgroundColorHandler = event -> {
    lastSelectedTabIndex = tabContainer.getSelectionModel().getSelectedIndex();

    Tab currentTab = (Tab) event.getTarget();
    if (currentTab.isSelected()) {
        currentTab.setStyle("-fx-background-color: -fx-focus-color;");
    } else {
        currentTab.setStyle("-fx-background-color: -fx-accent;");
    }
};

/// 8.
EventHandler<Event> logoutHandler = event -> {
    Tab currentTab = (Tab) event.getTarget();
    if (currentTab.isSelected()) {
        tabContainer.getSelectionModel().select(lastSelectedTabIndex);

        // TODO: logout action
        // good place to show Dialog window with Yes / No question
        System.out.println("Logging out!");
    }
};
```

Ad. 7 - `replaceBackgroundColorHandler` will be used for normal case scenario when we have a tab menu and corresponding content.

Ad. 8 - `logoutHandler` is for a **logout** action and its main task is to make sure we didn't switch the selected tab and trigger whatever we want to do here. In my project, I am showing here a YES / NO dialog option.

We pass the `EventHandler` to the `configureTab()` function so it should look like this:

```
configureTab(userProfileTab, "User\nProfile", "/co/synappse/project01/resources/images/user-profile.png", replaceBackgroundColorHandler);
configureTab(settingsTab, "Settings", "/co/synappse/project01/resources/images/settings.png", replaceBackgroundColorHandler);
configureTab(logoutTab, "Logout", "/co/synappse/project01/resources/images/logout.png", logoutHandler);
...

private void configureTab(Tab tab, String title, String iconPath, EventHandler<Event> onSelectionChangedEvent) {
    ...
    tab.setOnSelectionChanged(onSelectionChangedEvent);
}
```

Then we need to make an initial selection of the first tab. Because the selected and unselected state is done manually it won't be done automatically when we will open the app for the first time.

```
userProfileTab.setStyle("-fx-background-color: -fx-focus-color;");
```

### Add content views

I didn't want to do everything in one fxml file, so I've decided to dynamically load content for each tab. In order to do that we need to store a reference to the `AnchorPane`s located inside each `Tab` which will hold the view where we will inject our content.

For this, to work, we need to add few lines of code. Starting off with references to tab's `AnchorPane`.

```
@FXML
private AnchorPane userProfileContainer;

@FXML
private AnchorPane settingsContainer;
```

Then we will modify the `configureView()` by adding there a `URL` with a fxml file and `AnchorPane` with reference to container view.

```
private void configureTab(Tab tab, String title, String iconPath, AnchorPane containerPane, URL resourceURL, EventHandler<Event> onSelectionChangedEvent) {

        ....

        if (containerPane != null && resourceURL != null) {
            try {
                Parent contentView = FXMLLoader.load(resourceURL);
                containerPane.getChildren().add(contentView);
                AnchorPane.setTopAnchor(contentView, 0.0);
                AnchorPane.setBottomAnchor(contentView, 0.0);
                AnchorPane.setRightAnchor(contentView, 0.0);
                AnchorPane.setLeftAnchor(contentView, 0.0);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
}
```

And use it in its final form

```
configureTab(userProfileTab, "User\nProfile", "/co/synappse/project01/resources/images/user-profile.png", userProfileContainer, getClass().getResource("userprofile.fxml"), replaceBackgroundColorHandler);
configureTab(settingsTab, "Settings", "/co/synappse/project01/resources/images/settings.png", settingsContainer, getClass().getResource("settings.fxml"), replaceBackgroundColorHandler);
configureTab(logoutTab, "Logout", "/co/synappse/project01/resources/images/logout.png", null, null, logoutHandler);
```

Full source code for this can be found on [GitHub](https://github.com/Synappse/tutorials/tree/master/javafx/01-vertical-stateful-tabbar-with-icons).