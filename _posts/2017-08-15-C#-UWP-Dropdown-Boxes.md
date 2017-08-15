---
layout: post
title: Loading Data into C# UWP Drop Down Boxes
tags: [C#, UWP, Programming]
---

Whilst working on a project recently I found the need to reload values back into a UWP Drop Down box using C#. The program is essentially a digital form, where users enter in details into text boxes and make selections using the drop down boxes. I wanted the ability for users to be able to save a form and then reload it at a later time. This was easily achieved and a delimited file was created housing all the values.

To load the data in, I simply broke the delimited file into an array and then started assigning values to the various fields. Except once I got to my drop down boxes, there was no easy way to assign a value. This makes sense as a drop down box should only be able to select from its pre-defined values, however the values I was entering came directly from this drop down box. 

After a few hours of searching I finally found a clue in an obscure message board that I can no longer remember. However the key to getting this down is simple. You loop through the values of your drop box, for each value you perform a check, does the value equal the value you are wishing to enter. If so, assign the value of the drop box as that item. Here's the code: 

{% highlight c# %}
foreach (var item in dropbox.Items)
  {
    if ((item as ComboBoxItem).Content.Equals(data[0]))
      {
        dropbox.SelectedItem = item;
      }
  }
{% endhighlight %}

In the above code snippet, to provide some context, dropbox is our Drop Box and the array data[0] is the data in which we have taken from our delimited saved file. 

Simple but effective.

Cheers,
