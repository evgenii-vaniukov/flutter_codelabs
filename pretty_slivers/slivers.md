# Step 1
## Introduction
Welcome to the introductory sliver workshop for Flutter! 👋

In this workshop, you will learn how to efficiently build scrolling widgets and use slivers directly, to create a rich scrolling experience for users.

This workshop is best for folks already familiar with building simple Flutter applications. If this is your first time using Flutter, then check out Writing Your First Flutter App to get started using Flutter.

### What is a sliver?
A sliver is a portion of a scrollable area, **which means anything that scrolls in Flutter is a sliver**. If you've used ListView or GridView, congratulations! You've already used a sliver in your Flutter app. Most often, slivers are wrapped in convenience classes such as these. This is because *slivers use a different layout protocol from most other widgets in the Flutter framework.* We'll discuss layout protocol later in the workshop. For now, let's take a tour of our starter code.

### Simple scrolling UI
Here, we have a forecasting app, called Horizons.

This MaterialApp consists of a Scaffold with a WeeklyForecastList as the body. This widget consists of a SingleChildScrollView that contains a Column featuring the next 7-day forecast.

*We also have our own ScrollBehavior set for our app in the MaterialApp().*

MaterialApp(
  scrollBehavior: ConstantScrollBehavior(),
  // ...
)
ScrollBehaviors are inherited by descendent Scrollables. They inform a Scrollable's ScrollPhysics and apply decorations like Scrollbars and GlowingOverscrollIndicators.

*By default, ScrollBehaviors are dynamic and change depending on the current platform that you're running this workshop on, like ScrollPhysics.* **On Mac and iOS platforms, you'll see BouncingScrollPhysics by default, while on others you'll see ClampingScrollPhysics.** To provide a consistent experience for everyone taking this workshop, we added a custom ScrollBehavior.

Where to next?
Throughout this workshop, we'll build on this code. We’ll convert the SingleChildScrollView to be a more efficient, lazy loading ListView.This allows us to add more to our app and remain performant.

We’ll then dive a little deeper and transform the code to use slivers directly, and we’ll experiment with dynamic scrolling effects like floating app bars. Move on to the next step to begin!

# Step 2

## Efficient scrolling in Flutter
The first task is to convert the SingleChildScrollView to make our scrolling UI more efficient, but first, let's discuss why and when this widget might not be the most efficient choice.

Currently, the UI is pretty simple. The scrolling 7-day forecast is likely to fit most screens. This is OK for now, but as we add to the UI, we want to remain performant.

If we remove the SingleChildScrollView, then we could get an error due to the Column overflowing. If you want to, try and see what this looks like by resizing the window so that the contents of the Column don’t fit in the screen.

*This is where we can see how the layout protocol of slivers differs from their relative, the box layout protocol. A Column is laid out using BoxConstraints. It has a height and width, and a position within the window. A Column cannot lay out beyond the bounds of the window.*

*When we wrap the Column in a SingleChildScrollView, we’re essentially wrapping the Column in a sliver. Slivers lay out using SliverContraints and have a SliverGeometry. When you work with slivers, the window size that we were constricted to previously becomes an infinite amount of space in the given axis.* So, we use language different from height, width, and position. For a sliver, we need to know things like how much is visible, how far to the next sliver, and how far we scrolled. **The answers enable us to lazily load slivers, meaning we only build slivers that we can see and a little bit of the ones on either edge.** This makes scrolling more efficient because we won’t build slivers that we don’t need because the slivers aren’t seen.

Making the switch
**Because a SingleChildScrollView is only one sliver, we aren’t lazily loading the UI. Instead, as the Column gets bigger, all of its contents are built.** Before we add more to the UI, let’s make it more efficient by using ListView.

We’ll use the builder constructor, which is called on demand, for lazy loading.

Replace the SingleChildScrollView with the ListView.builder, and remove the Column. This constructor requires an itemBuilder, which provides a BuildContext and the current item index. We must also specify the itemCount because we know that we’re displaying a weekly forecast.

return ListView.builder(
  itemCount: 7,
  itemBuilder: (BuildContext context, int index) {
    return Card(
      // This remains unchanged.
    );
  }
);
**Now each of the Cards is wrapped in a sliver and only built as it’s needed.**

We’re also going to change the access to the mock Server class. Rather than request all of the data at one time, we can access the forecasts by index in our builder.

// Remove this:
final List<DailyForecast> forecasts =
    Server.getDailyForecastList();

// And add this in our ListView itemBuilder
final DailyForecast dailyForecast =
    Server.getDailyForecastByID(index);

Now that we’re building more efficiently, let’s add more to the ListView.

# Step 3
## Adding To Our UI
Now that we’re lazily building the UI, let’s add some complexity. The DailyForecast object comes with an image for each day, but we can polish this a bit more.

Currently, we’re using a ListTile in the Card. This is a handy widget that handles a lot of layout and padding for you. Let’s change the widget to reflect our own style. Feel free to diverge here. This is a fun side mission to add to the UI now that it’s more efficient.

If we look at the current UI, then we can break it into Columns and Rows, and make a few adjustments.

Let’s remove the ListTile and create a Row instead. The contents of leading and trailing can become children of the Row. Because title and subtitle are stacked vertically, let’s wrap them in a Column and place the Column in between.

Row( 
  children: <Widget>[
    // The former contents of our ListTile:
    // leading
    Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: <Widget>[
          // title
          // subtitle
      ],
    )
    // trailing
  ],
)
This is pretty close to what the ListTile generated. Add some Padding to neaten the UI, **and put an Expanded widget around the Column to handle any overflow from the forecast description.**

Expanded(
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: <Widget>[
      // title
      // subtitle
    ],
  )
)

# Step 4
## Working directly with slivers
This is looking really nice now. Although, the AppBar seems a little less exciting. Let’s work with slivers directly, so we can experiment with the SliverAppBar.

First, we need to do some setup to work with slivers. *Slivers are contained in a ScrollView.* The ListView set up some of this for us, so now we need a container for the slivers. Let’s use a CustomScrollView. Go ahead, and place one in the body of the Scaffold. A custom ScrollView takes a list of slivers, so place the WeeklyForecastList there, and we’ll convert that next.

home: Scaffold(
  appBar: AppBar(
    title: Text('Horizons'),
    backgroundColor: Colors.teal[800],
  ),
  body: CustomScrollView(
    slivers: <Widget>[
      WeeklyForecastList(),
    ],
  ),
),
Now, for the WeeklyForecastList, let’s change it to use a SliverList instead of a ListView. A SliverList takes a SliverChildDelegate, which provides the children. One kind of SliverChildDelegate is a SliverChildBuilderDelegate, which is the same as the builder provided in the ListView. We can keep the existing builder, and provide it to the delegate instead. The delegate also takes a childCount, which is similar to the ListView’s itemCount.

return SliverList(
  delegate: SliverChildBuilderDelegate(
    (BuildContext context, int index) {
      final DailyForecast dailyForecast =
          Server.getDailyForecastListByID(index);
      return Card(
        // Remains the same
      );
    },
    childCount: 7,
  )
);
You are now working **directly** with slivers - hooray! 🎉

# Step 5
## Adding a SliverAppBar
Now that we have our CustomScrollView and SliverList in place, we can add a SliverAppBar for a more dynamic header.

In the HorizonsApp, add a SliverAppBar at the top of the CustomScrollView, and remove the AppBar in the Scaffold. The SliverAppBar shares similar properties with the existing AppBar, so let’s migrate those too.

body: CustomScrollView(
  slivers: <Widget>[
    SliverAppBar(
      title: Text('Horizons'),
      backgroundColor: Colors.teal[800],
    ),
    WeeklyForecastList(),
  ],
),
We now have a scrolling app bar. This app bar scrolls as if it were part of our list. The SliverAppBar has a lot more dynamic features though, so let’s explore pinned, floating, snap, and collapsing behaviors for the new SliverAppBar.

Pinning the app bar keeps it at the top of the screen, like the AppBar we had before.

SliverAppBar(
  title: Text('Horizons'),
  backgroundColor: Colors.teal[800],
  pinned: true,
),
A floating SliverAppBar will scroll out of view, but it scrolls back into view when the user scrolls back in that direction, regardless of the current position in the scroll view.

SliverAppBar(
  title: Text('Horizons'),
  backgroundColor: Colors.teal[800],
  floating: true,
),
Floating app bars also support snap animation. This animation snaps the SliverAppBar in and out of view as the user scrolls, rather than floating in with the user input.

SliverAppBar(
  title: Text('Horizons'),
  backgroundColor: Colors.teal[800],
  floating: true,
  snap: true,
),
Last, let’s see how the SliverAppBar behaves when we use an expandedHeight. This adds to the size of the app bar and collapses as the user scrolls. The expandedHeight can be combined with the floating, pinned, and snap features that were discussed earlier.

SliverAppBar(
  title: Text('Horizons'),
  backgroundColor: Colors.teal[800],
  pinned: true,
  expandedHeight: 200.0,
),
We can fill this extra space using a FlexibleSpaceBar. Let’s explore that in the next step.

# Step 6
## Flexible space in a SliverAppBar
We can **fill the expandedHeight of our app bar**, for an even more dynamic UI.

We can do this with the **FlexibleSpaceBar.** *This widget is designed to stretch and collapse its content. Let’s try it out!*

Add a FlexibleSpaceBar to your SliverAppBar. You can move the title here and add an image as the background.

SliverAppBar(
  pinned: true,
  backgroundColor: Colors.teal[800],
  expandedHeight: 200.0,
  flexibleSpace: FlexibleSpaceBar(
    title: Text('Horizons'),
    background: Image.network(
      headerImage,
      fit: BoxFit.cover,
    ),
  ),
),
Now, as the user scrolls, the app bar collapses with a parallax effect on the image and fades into the pinned SliverAppBar after it’s fully collapsed. The FlexibleSpaceBar supports multiple CollapseModes, with this parallax behavior being the default. You can also try out pin and none to see how the effect can vary.

FlexibleSpaceBar(
  title: Text('Horizons'),
  collapseMode: CollapseMode.pin,
  // ... 
),
For better contrast, let’s add another gradient effect behind the title. We can do this as we did before, but let’s use a LinearGradient instead.

Wrap your Image in aDecoratedBox, with the LinearGradient in the DecorationPosition.foregound.

FlexibleSpaceBar(
  title: Text('Horizons'),
  background: DecoratedBox(
    position: DecorationPosition.foreground,
    decoration: BoxDecoration(
      gradient: LinearGradient(
        begin: Alignment.bottomCenter,
        end: Alignment.center,
        colors: <Color>[
          Colors.teal[800]!,
          Colors.transparent,
        ],
      ),
    ),
    child: Image.network(
      headerImage,
      fit: BoxFit.cover,
    ),
  ),
),
Just as the FlexibleSpaceBar supports collapsing our expanded SliverAppBar, it also works hand-in-hand with another SliverAppBar feature - **stretch**. Let's see how stretching can add a nice finishing touch to our app, as well as some extra functionality.

# Step 7
## Stretching the FlexibleSpace
In order to stretch our SliverAppBar, we must be using BouncingScrollPhysics. This is part of the ConstantScrollBehavior we had set up in the very beginning of this workshop. BouncingScrollPhysics allows us to overscroll, revealing empty space at the end of a scrollable area.

With SliverAppBar, we can stretch to fill this empty space, creating a cool visual, while also providing a callback. This feature is often used to reload data - the weather is constantly changing after all - so let's see how we can use this to refresh our forecast.

First we'll want to set SliverAppBar.stretch to true. We can provide a callback to onStretchTrigger, this will be called when the user overscrolls far enough - with far enough being 100 pixels by default. You can customize this further with stretchTriggerOffset. This callback is where we can refresh our data or complete another operation.

SliverAppBar(
  pinned: true,
  stretch: true,
  onStretchTrigger: () async {
    print('Load new data!');
    // await Server.requestNewData();
  }
  //...
Now we can pull down from the top edge of our scroll view and see the FlexibleSpaceBar stretch into the overscroll space, and if we check the console, we can see our print statement from our callback.

The FlexibleSpaceBar can add a few finishing touches here too. By default, the FlexibleSpaceBar has a StretchMode that will transform the background to fill the overscrolled space. There are other StretchModes though, and you can combine them for neat effects - you can have the title's Opacity fade as the SliverAppBar stretches, and apply a blur to the background as well.

Feel free to try these out and see what you like best as you stretch.

FlexibleSpaceBar(
  stretchModes: <StretchMode>[
    StretchMode.zoomBackground,
    StretchMode.fadeTitle,
    StretchMode.blurBackground,
  ],
  // ...
Let's review everything that we have done in this workshop.


