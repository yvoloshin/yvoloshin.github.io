---
layout:     post
title:      When Bootstrap Grid Is Not Enough 
date:       2017-10-15 20:00:00
author:     Yury Voloshin
summary:    Making a page more responsive with jQuery
categories: Twitter Bootstrap, jQuery
---
This is the second post related to my side project [RoutineTrack](http://www.routinetrack.com), where I will talk about the challenge of making a form truly responsive. The form in question is the form for creating a new workout routine. On a large desktop screen, the form looks like this:
[Imgur](https://i.imgur.com/7NC3on0.png)
[![new routine form](https://i.imgur.com/7NC3on0.png)](https://i.imgur.com/7NC3on0.png)
The routine is made up exercises, where each has a name, a number of reps, a number of sets, and a weight load. On a large screen, arranging these fields horizontally works well. But on a mobile phone, this becomes a problem, since scrolling horizontally on a phone through several fields is really annoying. To deal with this, I decided to arrange some of the exercise fields vertically, to make it look like this on a small screen:
[![new routine form](https://i.imgur.com/9pMMy0W.jpg)](https://i.imgur.com/9pMMy0W.jpg)
My first thought was to use [Bootstrap grid](https://getbootstrap.com/docs/3.3/css/#grid-options) to change the width of exercise fields, depending on the screen size. This worked really well, so that the fields would transform from the first version to the second version at different screen sizes. But then I realized that this is not going to work because of the table header. The large-screen version of the form is arranged as one large [Bootstrap table](http://getbootstrap.com/docs/4.0/content/tables), with a header row that's aligned with the table columns. The header was necessary in the large-screen table, but it looked out of place in the small-screen form. I needed a way to make the header disappear on a small screen and reappear on a big screen as the column widths change. This could not be done with Bootstrap grid alone. With the help of a few stackoverflow posts, I came up with the following solution. The large-screen form was given a class called “desktop”, the small-screen form was given a “mobile” class, and both forms were added to the [page](https://github.com/yvoloshin/Strength-tracker/blob/master/app/views/workout_types/new.html.erb) at the same time.
Then I added this code to the page:

```html
<div id="desktopTest" class="hidden-xs"></div>
```
```jquery
$( document ).ready(function() {
	if ($('#desktopTest').is(':hidden')) {
	    $('.desktop').attr("hidden", true);
	} else {
		$('.mobile').attr("hidden", true);
	}
});
```

The line of html is creating a `div` with a `hidden-xs` class, which is one of a group of [Bootstrap Responsive Utility](https://v4-alpha.getbootstrap.com/layout/responsive-utilities) classes that change the visibility of elements depending on the screen size.  Elements with a `hidden-xs` class stay hidden when the screen is extra small (xs) or smaller. This empty `div` acts as a test probe that passes information about the screen size to the jQuery script below. The script checks whether the empty `div` is hidden and, depending on the result, it hides either the large-screen form or the small-screen form. This is a relatively simple and convenient method for making your pages more responsive than they can be with Bootstrap grid alone. 
