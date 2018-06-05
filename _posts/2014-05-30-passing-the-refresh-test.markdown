---
layout: post
title:  "Passing The Refresh Test"
date:   2014-05-30 00:56:32
---

In this modern world of the web, refreshing a web page should not lose data. It should not make me lose my progress. Until I read [@eviltrout](https://twitter.com/eviltrout "@eviltrout on twitter")'s post about [The Refresh Test](http://eviltrout.com/2014/04/10/the-refresh-test.html "The Refresh Test by Robin Ward"), I didn't realise preventing data loss on a refresh was so easy. @eviltrout pointed towards persisting form data in localStorage. I took this direction and investigated just how easy this is to do for an average web form. This is what I found out.

## localStorage and sessionStorage
The first question I faced was should I use localStorage or sessionStorage? For a simple one page web form, sessionStorage is the best. sessionStorage persists for a single session in the web browser, and is removed when the browser tab or window is closed. This means for a single page form, I can refresh the page and my data will still be there!

localStorage by contrast will persist within a browser until it is removed. This means that a browser tab or window can be closed, and the localStorage data will still be there when you open the browser again.

One of the other considerations I took into account is the fact that anybody can open a browser, and potentially access the localStorage data. With this in mind, sessionStorage strikes me as the safer approach.

## The Solution
Once I decided on the technology I wanted to use, I set out to find a suitable tool for the job.

After a quick search, I discovered a jQuery Plugin called **jQuery Storage API** by Julien Maurel. Initially I was planning to use Vanilla.js, but for my small example I didn't want to re-invent the wheel, merely to see what was possible.

**jQuery Storage API** simply wraps the native HTML5 Web Storage  tools in beautifully simple API calls.

The first step is to initialise the sessionStorage object:

	var storage = $.sessionStorage;

Next up, we need to add our sessionStorage data. HTML5 Web Storage stores data in key value pairs. I decided that I wanted to store the data as it is typed. For this I trigger data storage on keyup, change, and blur events in the browser. 

	$('.storeValue').bind("keyup change blur", function() {
		storage.set(this.name, $(this).val());
	});

So this code will store every inputs value and name in key value pairs in the sessionStorage each time the value of the input changes. All I need to do is apply the .storeValue class to an input and it's name and value will be stored in my sessionStorage.

At this point we are storing input values as we type and change input values. But if I refresh my browser, it appears as if I still loss my data. So the next step is to load the data from sessionStorage on page load.

	$('.storeValue').each(function() {
		$(this).val(storage.get(this.name));
	});

The above code simply loops through all inputs with the .storeValue class, and pulls the value for each input from the sessionStorage and puts it into the inputs value.

Now if I enter some values and refresh my page, my data is still there! Mission Accomplished. Not quite yet…

If I submit this form, then revisit it within the same browser session, all of the sessionStorage data will still be there. This gives the impression that the form never submitted my information, so I should hit submit and try again. To prevent this poor user experience, we need to clear the sessionStorage for this form on submit.

	$('.submit').click(function(){ 
		storage.removeAll();
	});

This code very simply removes all sessionStorage data when the submit button is clicked. For my super simple example, this works as expected. But you may have validation happening on click, or you may want to wait for a response from the form submission before clearing this data. In this case you will probably want to remove the data on a different event.

For my example, I also wanted to provide the same experience for visitors submitting the form with their keyboard, so I used the code below to trigger the click event on hitting enter on the submit button.

	$('.submit').keydown(function(event){    
		if(event.keyCode==13){
			$('.submit').trigger('click');
		}
	});

You can [view the full demo here](https://acyphus.github.io/experiments/refresh-test/ "Passing The Refresh Test Demo") and [view the source here](https://github.com/ACyphus/sessionStorageDemo "sessionStorage Demo Source Code").

If you have any questions or would like to get in touch, you can find me [@ACyphus](https://twitter.com/ACyphus "@ACyphus on twitter").
