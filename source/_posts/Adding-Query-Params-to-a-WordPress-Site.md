---
title: Adding Query Params to a WordPress Site
date: 2017-05-15 15:40:42
tags:
- tech
- WordPress
- Query Params
---

[WordPress](https://wordpress.com/ "WordPress's Homepage") can be a pain in the butt to work with, mostly because, for all that you can do (which albeit is a lot), you're best off doing it (as an old friend would say) within its guardrails––that is either with pre-baked components with input forms or with plugins (that might just break your website).

What was that?

![David Lynch says, "What?"](/images/lynch2.gif)

It powers 27% of the Internet.

OK. Custom code be damned. (But not really.)

What follows is a super simple implementation of vanilla Javascript on a WordPress site.

## Background
During a recent project, the client (an foreign-language-immersion daycare) wanted their WordPress site to show different prices for different locations, but they didn't want it to be obvious that different prices we're being shown for the same class. (They didn't want people asking to get the lower price.)

## The Query
The first thing to do is add the query to link. This was easy enough in this case for each unique class there was a unique button to a learn more page that covered the different classes for that age group across all their locations. So the links already looked like:
```
...programs/infants/
```
all I had to do was add the query param:
```
...programs/infants/?location=enfield
```
I was able to hard code it in because the client had a table with all their classes separated by locations.

## The Javascript
First things first, we'll have to lay the groundwork for the scripts. I'm going to create an object with called `prices` and breakdown the different products (in my case number of days at daycare per week) by location, that will look something like:
```javascript
const prices = {
    threeDay: {
      enfield: '$825',
      boston: '$799',
      tucson: '$815'
    },
    fourDay: {
      enfield: '$1,010',
      boston: '$938',
      tucson: '$999'
    },
    fiveDay: {
      enfield: '$1,090',
      boston: '$1,065',
      tucson: '$1,080'
    }
  }
```

And then I'm also going to find the elements on the page (in this case the pricing table) where I'll be populating with information driven by the query parameter.
```javascript
const threeDay = document.getElementById('three-day');
const fourDay = document.getElementById('four-day');
const fiveDay = document.getElementById('five-day');
```

Now the interesting part: since the query param is in the link, we have to find it and parse it, then display information on a pricing table that's driven by it. Thankfully, WordPress allows Custom JS to be added to pages. So first step get the param:
```javascript
function getUrlParameter(name) {
    name = name.replace(/[\[]/, '\\[').replace(/[\]]/, '\\]');
    var regex = new RegExp('[\\?&]' + name + '=([^&#]*)');
    var results = regex.exec(location.search);
    return results === null ? '' : decodeURIComponent(results[1].replace(/\+/g, ' '));
  };
```
When the function is passed 'location' as an argument it will return the results, which in the case of `...programs/infants/?location=enfield`, results will be enfield.

Once I have that, I can create a function that will take that location as an arugment and sort through my `prices` object and find that location, something like:
```javascript
function definition1(location) {
    location = location || 'enfield'
    return prices.threeDay[location];
  }
```
Do that for each of the 'products' and now you have only to write the information to the page. The safety net above is that I'm setting the location to equal the location or `enfield`, estentially setting a default value in case someone comes to the page without a query param in the URL.

Now I just have to tie it all together:
```javascript
document.addEventListener("DOMContentLoaded", function(event) {
    threeDay.innerHTML = definition1(getUrlParameter('location'));
    fourDay.innerHTML = definition2(getUrlParameter('location'));
    fiveDay.innerHTML = definition3(getUrlParameter('location'));
  });
```
As you can see, I take the variable that I created earlier with the element's id and append the DOC with innerHTML––the `getUrlParameter` function returns a location, the `definition` functions use that returned location to return a string from the `prices` object with the price for the location that the user had clicked on.

To add all this to a WordPress page, you'll have to wrap all your code in `<script>` tags as if it were all inline Javascript because––WordPress. And while it could make sense to abstract it more (i.e. not hardcode functions to find a value in an object, or abstract the object one more and include class types like Infant, Toddler etc.) keeping the Javascript separated by page keeps the WordPress feel––I could theoretically share how to update the object with the non-technical client if their prices change in the future.

![David Lynch approves](/images/lynch1.gif)
<center>(David Lynch approves!)</center>

## Results/ Implications
A certain drawback to this implementation was that there was really no way a user was going to be able to tell that the pricing table on a classes page was different depending on which link they came from. While this is what the client wanted, it could misguide a user and lead to an awkward and poor conversation with a potential customer. As in writing, you want to pretend like you're writing to your most intelligent friend. Willfully keeping your friend ignorant may not be the best decision.
