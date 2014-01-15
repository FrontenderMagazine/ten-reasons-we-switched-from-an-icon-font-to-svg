# Ten reasons we switched from an icon font to SVG

We use **a lot** of icons on [lonelyplanet.com][1] and recently went through the
task of transferring them from an icon font to SVG files. I wanted to share why
we did this along with some of the current limitations to SVG and how we got
around them.

### 1. Separation of concerns.

We use a custom font on [lonelyplanet.com][2] and we used to bundle the icons
into the same file, storing the glyphs within the Private Unicode Area. This was
great because it meant one less HTTP request to fetch the icons but we also felt
that it limited our flexibility in how we prioritised resource loading.

We consider a small subset of our icons to be critical to the user's experience
but we don’t think that way about our font or the other icons. We try to load
non-critical assets after the page content and this was something we weren't
able to do with the previous approach.

Breaking the critical icons out from the font and the rest of the icons allowed
us to be more granular in how we delivered them to the user.

#### Counter argument

You don't have to bundle the font and the font icons together, you could serve
two separate fonts. This is something we probably would have done had we stuck
with the font-face solution.

### 2. Some devices don't honour the Private Unicode Area.

I'd heard rumours about devices overriding glyphs in the private unicode area
and using them to serve emoji but I hadn't seen it happen until recently. Emoji
was historically stored in the private unicode area, but at different ranges, so
it would make sense that there could be conflicts.

I can't remember which device I was testing on but I saw one of our tick icons
replaced with a multi-colour printer. The page looked amateurish and broken and
certainly gave us impetus to make this transition.

### 3. Black squares and crosses on Opera Mini.

Font face support and detection is historically quite tricky to get right. I'm
sure you've all seen this image of font awesome rendering on Opera Mini:

![example][font awesome rendering on Opera Mini]

I won't go over the intricacies of this problem as Opera Mini support and many
other platforms have been covered very well in [this article][3] by
[@kaelig][4].

I think what is really important though, beyond Opera Mini, is that this
highlights a blind spot that we're not able to control. We physically aren't
able to test on every device so it makes sense to favour techniques that are
more likely to render consistently.

We don't get a huge amount of traffic from Opera Mini at the moment but we're a
travel company serving people in all conditions on all bandwidths so we want to
do better than that. With SVG and PNG we feel more confident that users won't
get a broken and confusing page.

### 4. Chrome support for font-icons has been terrible recently.

Chrome Canary and Beta were hit with a fairly horrible [font bug][5] recently.
If you haven't yet noticed the bug, fonts have been unloading and reverting to a
system font after the page has experienced a period of inactivity.

When a font unloads and you're left with the text served as Georgia it can be a
little annoying. The page is still very usable though. If the same font is
responsible for serving the icons then suddenly the page is littered with black
squares and looks broken.

This bug was introduced during our transition to SVG. It was a relief to cut
over just as we were starting to get our first bug reports about it.

#### Counter argument

Those bugs haven't made it to a stable build of Chrome.

### 5. Crisper icons in Firefox.

We've found that our font renders at a slightly stronger weight in Firefox than
in other browsers. This is OK for text (although not great) but for icons it can
make the entire page look a bit unloved and clumsy. By using SVG we are able to
normalise the look and feel of our icons cross browser.

### 6. You don't always have to use generated content.

If you want to use font-icons in css you need to declare them using the content
property on generated content. Sometimes you might find yourself having to
complicate your code to make this possible i.e. because you are already using
the :before and :after pseudo elements or because the element doesn't support
generated content.

In that case you could choose to render it inline but you then end up with html
entities scattered through your markup which can easily be lost or forgotten
within a large application.

This problem is removed with SVG as you are not limited to generated content and
can render them as a background image on any element.

### 7. Less fiddly to position.

Admittedly this may be a result of how we created and managed our icon glyphs
but we always found them awkward to position exactly how we wanted (and in a
consistent fashion cross browser). We resorted to line height hacks and
absolute/relative positioning to get them just right and it was difficult to
come up with an abstraction that worked consistently.

With SVG we've found the placement much more willing. We use background-size:
cover and simply resize the element to ensure consistency across browsers.

### 8. Multi-colour icons.

Font icons are well known to have a single colour limitation. SVGs, on the other
hand, can support multiple colours as well as gradients and other graphical
features.

We have always had to support multi-colour icons for our maps and had previously
used an additional PNG sprite alongside our icon font. As a result of the move
to SVG we were able to delete this sprite which meant one less request for the
user and one asset fewer to maintain.

#### Counter argument

Multi-colour font icons can be accomplished using icon layering.

It is significantly more challenging to do so successfully though: if
positioning one glyph correctly cross-browser is tricky, it doesn't get easier
with two.

### 9. SVGs allow us to use animation within our icons.

We haven't yet utilised this feature but is likely something we will look into
now that we have made the jump.

#### 10. It's always felt like a hack.

Through a combination of all of the above, using font-face for icons has always
felt like a hack. It is a brilliant hack, no doubt, but it's still a different
asset type masquerading and being manipulated into something greater.

## What about the benefits of font-face?

Serving icons through font-face does have some benefits over SVG and we had to
consider these in depth before making the transition. The most pertinent
benefits for font-face are browser support and colour flexibility.

It's fair to say that it would have been considerably harder to tackle these
problems without the great work already done by the filamentgroup on
[Grunticon][6].

### Colour variations

The **huge** benefit to using an icon font is its flexibility. You have no
limitation to the amount of colour variations and can easily switch it depending
on the current state (:hover, :focus, .is-active etc.). This is a rare luxury
and very useful for quick development. It was also the reason we resisted making
the leap to SVG for so long.

#### Our solution

There are a few solutions out there to provide this functionality although all
of them have their own limitations (for now). We finally came up with a
technique which we were pretty happy with and which toed the line between
flexibility and resource size.

Grunticon is designed to declare each icon individually, thus avoiding having to
use sprites. We followed suit with this approach but, although we had one css
selector per icon, we served each icon in six different colours as shown here.

![icons][icon in six different colours]

As we were just duplicating the same icon multiple times within the same file,
it compressed down to the size of just one icon (plus around 50 bytes for gzip
pointers). This meant that we could have n colour variations for each icon at
almost no cost. Here's a [jsFiddle][7] showing how this technique works.

With this solution we had the flexibility back that we thought we would miss. By
taking away total flexibility it also brought the added benefit of reinforcing
colour palette consistency, something that had gradually been lost from our font
icon implementation.

With this technique it is also easy to apply state-based changes to the icons by
updating their background position.

It's worth mentioning that in the future we may be able to remove the sprite
altogether and use [SVG Fragment Identifiers][8] to change the colour.

### Browser support

Font icons [work all the way back to IE8][9] whereas SVG [does not][10]. For our
implementation we also required support for background-size although this is
[fairly comparable to SVG support][11].

#### Our solution

[Grunticon][12] handles legacy support right out of the box. It will create PNG 
versions of your SVG files and serve them to legacy browsers depending on 
whether or not they have support for SVG.

We decided to serve just a subset of the icons (the critical ones, like the
logo) to legacy browsers due to a lack of support for background-size. We could
have avoided the need for this css property by resizing all the original SVG
files but, as many of the icons are used at multiple sizes, we preferred to
serve just the critical icons and keep them consistent within the sprite.

We also tried using this [background-size polyfill][13] but quickly abandoned
it. I would **definitely recommend avoiding this** if you have more than one or
two images you need to resize. We found that more than two caused IE8 to crash
consistently.

## Was it worth it?

Both techniques are resolution independent, scalable and fairly lightweight so
if you are using either it is good for the user. We felt that on balance SVGs
gave us more confidence in how our application would appear to each user and
that extra element of control was ultimately what it came down to.

SVGs have been live on [Lonely Planet][14] since November 2013 and development
has been painless so far.

[1]: http://www.lonelyplanet.com/france/paris
[2]: http://www.lonelyplanet.com/france/paris
[3]: http://blog.kaelig.fr/post/33373448491/testing-font-face-support-on-mobile-and-tablet
[4]: https://twitter.com/kaelig
[5]: https://code.google.com/p/chromium/issues/detail?id=236298&q=font&colspec=ID%20Pri%20M%20Iteration%20ReleaseBlock%20Cr%20Status%20Owner%20Summary%20OS%20Modified
[6]: https://github.com/filamentgroup/grunticon
[7]: http://jsfiddle.net/Wmcbz/
[8]: http://www.w3.org/TR/SVG/linking.html#LinksIntoSVG
[9]: http://caniuse.com/#search=font-face%20web
[10]: http://caniuse.com/#search=SVG%20in%20css
[11]: http://caniuse.com/#search=background-size
[12]: https://github.com/filamentgroup/grunticon
[13]: https://github.com/louisremi/background-size-polyfill
[14]: http://www.lonelyplanet.com/england/london

[font awesome rendering on Opera Mini]: img/grunticon_workflow_operamini2.png
[icon in six different colours]: img/icon-example.png