---
layout: post
title: "ExtJS grid with WAI-ARIA support"
date: 2012-06-10 14:06
comments: true
categories: [Accessibility, Web, WAI-ARIA, ExtJS] 
---

[ExtJS](http://www.sencha.com/products/extjs/) is a great client side JS framework swith superior UI/widgets support. However comparing to other JS frameworks, such as [jQuery](http:///www.jquery.com), [ExtJS](http://www.sencha.com/products/extjs/) still does not have good support for [web accessibility](http://en.wikipedia.org/wiki/Web_accessibility) ([1](http://www.w3.org/WAI/)), or [508](http://www.section508.gov/). It has been [pushed back](http://www.sencha.com/forum/showthread.php?167849-Web-Accessibility-support&s=b0bf08edc0082a89203546ada761ae72) from release to release. I personally doubt it will be done in upcoming 4.2.

Although ExtJS includes an aria grid example in its 4.1 release, it sounds to me is just a theme trick with different color. No screen reader can recognize the table on the page. Neither [Mac VoiceOver](http://www.apple.com/accessibility/voiceover/) nor [JAWS](http://www.freedomscientific.com/products/fs/jaws-product-page.asp) can.

<!--more -->

This was exactly the tough work I got involved recently. If we can not fix the screen reader issue, customers will esclate us to death :). There is no way to roll back ExtJS grid to a standard HTML ```<table>``` implementation. The only approach left is [WAI-ARIA](http://www.w3.org/WAI/intro/aria.php). 

My frist attempt was to add these table related [roles](http://www.w3.org/TR/wai-aria/roles) in the grid DOM structure, such as [```grid```](http://www.w3.org/TR/wai-aria/roles#grid), [```row```](http://www.w3.org/TR/wai-aria/roles#row), [```columnheader```](http://www.w3.org/TR/wai-aria/roles#columnheader) and [```gridcell```](http://www.w3.org/TR/wai-aria/roles#gridcell). However, [JAWS](http://www.freedomscientific.com/products/fs/jaws-product-page.asp) reported there were two empty tables on the page not one table with data.

The DOM structure of ExtJS 4 grid looks like following:
``` html ExtJS 4 Grid DOM structure
<div id="gridpanel" role="grid"> <!-- the out grid div, where I put role=grid -->
	<div id="headercontainer" role="row"> <!-- the grid header container, where I put role=row -->
		<div id="headercontainer-inner">
			<div id="headercontainer-targetEl">
				<div id="gridcolumn-1" role="columnheader"> <!-- the grid column header where I put role=columnheader-->
				</div>
				<div id="gridcolumn-2"> <!-- the grid column header -->
				</div>
			</div>
		</div>
	</div>
	<div id="gridpanel-body">
		<div id="gridview">
			<table>
				<tbody>
					<tr class="x-grid-row" role="row">
						<td class="x-grid-cell" role="gridcell"> </td>
						<td class="x-grid-cell" role="gridcell"> </td>
					</tr>
				</tbody>
			</table>
		</div>
	</div>
</div>

```

After some digging, reading and googling, I suddenly realized screen reader like [JAWS](http://www.freedomscientific.com/products/fs/jaws-product-page.asp) will see the ```<table>``` as a table as well. However in this ExtJS implementation, it is simply used for presentation purpose. So I added ```role="presentation"``` on the ```<table>``` element. And BANG! [JAWS](http://www.freedomscientific.com/products/fs/jaws-product-page.asp) recognized my table properly.

What is the lesson learnt here? Don't try to mix and match HTML element with standard semantic with WAI-ARIA roles.
