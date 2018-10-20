---
date: 2018-05-03
title: Syntax Highlighting With Visual Studio
weight: 10
draft: true
---

Typical syntax highlighters like `highlight.js` and `pygment` are only able to discern a limited amount of syntax information from standard text. Taking advantage of Visual Studio's syntax highlighting and the ability to [Copy as HTML](https://marketplace.visualstudio.com/items?itemName=VisualStudioPlatformTeam.CopyAsHtml) allows me to get full syntax highlighting in these blog posts.
Here's a quick example of the difference:

Using `highlight.js`

```cpp
template <typename F>
void register_rpc(HashedID name, F* func)
{
	static_assert(std::is_void<rpc<F>::return_type>::value,
		"You can't register a function as an RPC if it returns a non-void type");
	rpc<F> rpc;
	rpc.add_rpc(name, func);
}
```

Using `Visual Studio 2017`

<pre class="visualstudio"><span class="keyword">template</span>&nbsp;<span class="operator">&lt;</span><span class="keyword">typename</span>&nbsp;<span class="cppType - identifier - (TRANSIENT)">F</span><span class="operator">&gt;</span>
<span class="keyword">void</span>&nbsp;<span class="cppFunctionTemplate - identifier - (TRANSIENT)">register_rpc</span><span class="operator">(</span><span class="cppType - identifier - (TRANSIENT)">HashedID</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">name</span><span class="operator">,</span>&nbsp;<span class="cppType - identifier - (TRANSIENT)">F</span><span class="operator">*</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">func</span><span class="operator">)</span>
<span class="operator">{</span>
	<span class="keyword">static_assert</span><span class="operator">(</span><span class="cppNamespace - identifier - (TRANSIENT)">std</span><span class="operator">::</span><span class="cppType - identifier - (TRANSIENT)">is_void</span><span class="operator">&lt;</span><span class="cppType - identifier - (TRANSIENT)">rpc</span><span class="operator">&lt;</span><span class="cppType - identifier - (TRANSIENT)">F</span><span class="operator">&gt;::</span><span class="identifier">return_type</span><span class="operator">&gt;::</span><span class="identifier">value</span><span class="operator">,</span>
		<span class="string">&quot;You&nbsp;can&#39;t&nbsp;register&nbsp;a&nbsp;function&nbsp;as&nbsp;an&nbsp;RPC&nbsp;if&nbsp;it&nbsp;returns&nbsp;a&nbsp;non-void&nbsp;type&quot;</span><span class="operator">);</span>
	<span class="cppType - identifier - (TRANSIENT)">rpc</span><span class="operator">&lt;</span><span class="cppType - identifier - (TRANSIENT)">F</span><span class="operator">&gt;</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">rpc</span><span class="operator">;</span>
	<span class="cppLocalVariable - identifier - (TRANSIENT)">rpc</span><span class="operator">.</span><span class="identifier">add_rpc</span><span class="operator">(</span><span class="cppParameter - identifier - (TRANSIENT)">name</span><span class="operator">,</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">func</span><span class="operator">);</span>
<span class="operator">}</span></pre>

<!--more-->

The main difference (aside from the colour scheme) is that Visual Studio has full context information about the snippet, such as template types and alias's.

There are two options when wanting to use VS syntax. The first option is to just use the default settings where inline CSS colours are applied to the tags encapsulating the code. This makes it super simple to get started but the ability to change the syntax colours in the future is lost.

The second option is what I chose to do. I have `Copy As HTML` output CSS class's that I can then style myself. I did this by modifying some of the Copy As HTML settings and creating an appropriate CSS stylesheet. If you decide to go down this route you'll make your life so much easier if you edit the settings to output an additional class for the `pre` tag (I chose `visualstudio`), and to output syntax colours with classes so you can copy and paste the colours that get outputted and match them to the classes in your CSS file very quickly. Remember to make sure to disable the syntax colour output once you're done so that the inline CSS colours don't override your custom ones.

The biggest downside with using Visual Studio is that in order to have this contextual syntax information you need to have a valid Visual Studio project open for anything remotely complex in order to copy and paste the syntax highlighting, so it makes it impossible to just write generic code snippets directly in to these blog posts in markdown if you want the full syntax highlighting. It also makes it difficult to edit or update any code in your blog as you have to open up the specific VS project it was originally in and edit it there before copying it over (or recreate it). Trying to apply syntax highlighting manually would take a lot of work setting up the CSS classes as there's often a class every few characters.

The ideal situation is to have both options available. Use Visual Studio most of the time but fall back on standard syntax highlighters like `highlight.js` when you just need a quick code snippet. This is achievable, it just requires double the work as you'll need to modify the CSS of both syntax highlighters now to match eachother as closely as possible