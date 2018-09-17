---
date: 2018-05-05
title: A Class to Serialize and Deserialize Data
weight: 10
draft: true
---

In developing RPC capabilities for my dissertation I need to be able to serialize and deseriliaze binary data. I'll be making a basic `Packet` class capable of doing so, and then we'll look at expanding it and fixing any issues that come up.

We'll start with what some simple use of this class should look like:

<pre class="visualstudio"><span class="keyword">int</span>&nbsp;<span class="cppFunction - identifier - (TRANSIENT)">main</span><span class="operator">()</span>
<span class="operator">{</span>
	<span class="cppType - identifier - (TRANSIENT)">Packet</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">p</span><span class="operator">;</span>
	<span class="cppLocalVariable - identifier - (TRANSIENT)">p</span>&nbsp;<span class="operator">&lt;&lt;</span>&nbsp;<span class="number">5</span><span class="operator">;</span>
	<span class="keyword">int</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">five</span><span class="operator">;</span>
	<span class="cppLocalVariable - identifier - (TRANSIENT)">p</span>&nbsp;<span class="operator">&gt;&gt;</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">five</span><span class="operator">;</span>
	<span class="keyword">return</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">five</span><span class="operator">;</span>
<span class="operator">}</span></pre>

I did say simple.

<!--more-->

As you can tell we'll be using stream operator overloading rather than direct function calls. This has the advantage of being idiomatic (`std::cout`, `std::stringstream`, `std::fstream`) and of chaining serialization/deserialization together. The alternative would be something like `Packet.write_int(int i)` which has the advantage of being usable inside of a function call such as `f(p.read_int(), p.read_float())` but the downside of being verbose when needing to read or write lots of data.

So our packet class should look something like this

<pre class="visualstudio"><span class="keyword">class</span>&nbsp;<span class="cppType - identifier - (TRANSIENT)">Packet</span>
<span class="operator">{</span>
<span class="keyword">public</span><span class="operator">:</span>
	<span class="cppType - identifier - (TRANSIENT)">Packet</span><span class="operator">&amp;</span>&nbsp;<span class="cppMemberOperator - keyword - (TRANSIENT)">operator</span><span class="cppMemberOperator">&nbsp;</span><span class="cppMemberOperator - operator - (TRANSIENT)">&lt;&lt;</span><span class="operator">(</span><span class="keyword">int</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">i</span><span class="operator">)</span>
	<span class="operator">{</span>
		<span class="comment">//serialize</span>
		<span class="keyword">return</span>&nbsp;<span class="operator">*</span><span class="keyword">this</span><span class="operator">;</span>
	<span class="operator">}</span>
 
	<span class="cppType - identifier - (TRANSIENT)">Packet</span><span class="operator">&amp;</span>&nbsp;<span class="cppMemberOperator - keyword - (TRANSIENT)">operator</span><span class="cppMemberOperator">&nbsp;</span><span class="cppMemberOperator - operator - (TRANSIENT)">&gt;&gt;</span><span class="operator">(</span><span class="keyword">int</span><span class="operator">&amp;</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">i</span><span class="operator">)</span>
	<span class="operator">{</span>
		<span class="comment">//deserialize</span>
		<span class="keyword">return</span>&nbsp;<span class="operator">*</span><span class="keyword">this</span><span class="operator">;</span>
	<span class="operator">}</span>
 
<span class="keyword">private</span><span class="operator">:</span>
<span class="operator">};</span></pre>

Of course this doesn't do anything yet so let's focus on serializing the data. We'll need somewhere to store it so something like `std::array` would work just fine, but I'd like to be able to have packets that can grow, so `std::vector` suits my needs better. Note that both of these offer contiguous storage which is ideal for us.

How do we create a vector that can store anything in it though? We could use `std::vector<std::any>>` which would work fine when deserializing the data on the same machine but this Packet has to eventually be transformed in to a pointer and a length in order to send it across the network, so in our case we actualy want to store the binary data which is just a bunch of bytes. A byte can be presented by anything that has a size of 1 byte, which in the case of C++ is the `char` type. C++17 offers a `std::byte` though so we'll use that instead as it makes our intentions more clear and forces us to `static_cast` it to a `char` rather than be implicitly casted.

So now we have a `std::vector<std::byte>>` but how do we push data on to it? If we have an int (which is often 4 bytes) we could try and split it up in to 4 bytes and push each one on separately, or we can just grab its memory address `&i` and length `sizeof(i)`. `std::vector` provides us with access to the underlying contiguous storage through a call to `data()`. So now we have a place to store our bytes and a bunch of bytes to store, how do we actually go about doing it?

in C `memcpy` would be the standard way, in C++ we can use `std::copy` instead. Which one to use is up to you, you might think `memcpy` is faster as `std::copy` will often result in a `memcpy` but in some cases it can have minor performance improvements due to the compiler having knowledge of the type being copied rather copying a `void*`. I'll be using `memcpy` because I'll be working with raw pointers rather than iterators but changing this later to experiment with performance improvements would be inconsequential.

 The full code now looks like this:
 
<pre class="visualstudio"><span class="preprocessor keyword">#include</span>&nbsp;<span class="string">&lt;vector&gt;</span>
<span class="preprocessor keyword">#include</span>&nbsp;<span class="string">&lt;cstddef&gt;</span>
 
<span class="keyword">class</span>&nbsp;<span class="cppType - identifier - (TRANSIENT)">Packet</span>
<span class="operator">{</span>
<span class="keyword">public</span><span class="operator">:</span>
	<span class="cppType - identifier - (TRANSIENT)">Packet</span><span class="operator">&amp;</span>&nbsp;<span class="cppMemberOperator - keyword - (TRANSIENT)">operator</span><span class="cppMemberOperator">&nbsp;</span><span class="cppMemberOperator - operator - (TRANSIENT)">&lt;&lt;</span><span class="operator">(</span><span class="keyword">int</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">i</span><span class="operator">)</span>
	<span class="operator">{</span>
		<span class="cppMemberFunction - identifier - (TRANSIENT)">serialize</span><span class="operator">(&amp;</span><span class="cppParameter - identifier - (TRANSIENT)">i</span><span class="operator">,</span>&nbsp;<span class="keyword">sizeof</span><span class="operator">(</span><span class="cppParameter - identifier - (TRANSIENT)">i</span><span class="operator">));</span>
		<span class="keyword">return</span>&nbsp;<span class="operator">*</span><span class="keyword">this</span><span class="operator">;</span>
	<span class="operator">}</span>
 
	<span class="cppType - identifier - (TRANSIENT)">Packet</span><span class="operator">&amp;</span>&nbsp;<span class="cppMemberOperator - keyword - (TRANSIENT)">operator</span><span class="cppMemberOperator">&nbsp;</span><span class="cppMemberOperator - operator - (TRANSIENT)">&gt;&gt;</span><span class="operator">(</span><span class="keyword">int</span><span class="operator">&amp;</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">i</span><span class="operator">)</span>
	<span class="operator">{</span>
		<span class="comment">//deserialize</span>
		<span class="keyword">return</span>&nbsp;<span class="operator">*</span><span class="keyword">this</span><span class="operator">;</span>
	<span class="operator">}</span>
 
<span class="keyword">private</span><span class="operator">:</span>
	<span class="keyword">void</span>&nbsp;<span class="cppMemberFunction - identifier - (TRANSIENT)">serialize</span><span class="operator">(</span><span class="keyword">int</span><span class="operator">*</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">start</span><span class="operator">,</span>&nbsp;<span class="keyword">int</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span><span class="operator">)</span>
	<span class="operator">{</span>
		<span class="cppFunction - identifier - (TRANSIENT)">memcpy</span><span class="operator">(</span><span class="cppMemberField - identifier - (TRANSIENT)">bytes</span><span class="operator">.</span><span class="cppMemberFunction - identifier - (TRANSIENT)">data</span><span class="operator">(),</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">start</span><span class="operator">,</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span><span class="operator">);</span>
	<span class="operator">}</span>
 
	<span class="cppNamespace - identifier - (TRANSIENT)">std</span><span class="operator">::</span><span class="cppType - identifier - (TRANSIENT)">vector</span><span class="operator">&lt;</span><span class="cppNamespace - identifier - (TRANSIENT)">std</span><span class="operator">::</span><span class="cppType - identifier - (TRANSIENT)">byte</span><span class="operator">&gt;</span>&nbsp;<span class="cppMemberField - identifier - (TRANSIENT)">bytes</span><span class="operator">{</span><span class="number">10</span><span class="operator">};</span>
<span class="operator">};</span>
 
<span class="keyword">int</span>&nbsp;<span class="cppFunction - identifier - (TRANSIENT)">main</span><span class="operator">()</span>
<span class="operator">{</span>
	<span class="cppType - identifier - (TRANSIENT)">Packet</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">p</span><span class="operator">;</span>
	<span class="cppLocalVariable - identifier - (TRANSIENT)">p</span>&nbsp;<span class="cppMemberOperator - operator - (TRANSIENT)">&lt;&lt;</span>&nbsp;<span class="number">5</span><span class="operator">;</span>
	<span class="keyword">int</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">five</span><span class="operator">;</span>
	<span class="cppLocalVariable - identifier - (TRANSIENT)">p</span>&nbsp;<span class="cppMemberOperator - operator - (TRANSIENT)">&gt;&gt;</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">five</span><span class="operator">;</span>
	<span class="keyword">return</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">five</span><span class="operator">;</span>
<span class="operator">}</span></pre>

We start off with 10 bytes `std::vector<std::byte bytes{10}` and then memcpy the int to the beginning of the vector. Adding a breakpoint and inspecting the underlying data shows us that we've successfully stored the value of '5'.

![hi](/blog/images/packet/five.png)

If we kept serializing data we'd override the first four bytes rather than append to them since we don't keep track of how much we've written. Now let's get deserializing working. Rather than `memcpy` the `int` to our storage, we want to do the inverse.

<pre class="visualstudio">
<span class="cppType - identifier - (TRANSIENT)">Packet</span><span class="operator">&amp;</span>&nbsp;<span class="cppMemberOperator - keyword - (TRANSIENT)">operator</span><span class="cppMemberOperator">&nbsp;</span><span class="cppMemberOperator - operator - (TRANSIENT)">&gt;&gt;</span><span class="operator">(</span><span class="keyword">int</span><span class="operator">&amp;</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">i</span><span class="operator">)</span>
<span class="operator">{</span>
	<span class="cppMemberFunction - identifier - (TRANSIENT)">deserialize</span><span class="operator">(&amp;</span><span class="cppParameter - identifier - (TRANSIENT)">i</span><span class="operator">,</span>&nbsp;<span class="keyword">sizeof</span><span class="operator">(</span><span class="cppParameter - identifier - (TRANSIENT)">i</span><span class="operator">));</span>
	<span class="keyword">return</span>&nbsp;<span class="operator">*</span><span class="keyword">this</span><span class="operator">;</span>
<span class="operator">}</span></pre>

<pre class="visualstudio">
<span class="keyword">void</span>&nbsp;<span class="cppMemberFunction - identifier - (TRANSIENT)">deserialize</span><span class="operator">(</span><span class="keyword">int</span><span class="operator">*</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">start</span><span class="operator">,</span>&nbsp;<span class="keyword">int</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span><span class="operator">)</span>
<span class="operator">{</span>
	<span class="cppFunction - identifier - (TRANSIENT)">memcpy</span><span class="operator">(</span><span class="cppParameter - identifier - (TRANSIENT)">start</span><span class="operator">,</span>&nbsp;<span class="cppMemberField - identifier - (TRANSIENT)">bytes</span><span class="operator">.</span><span class="cppMemberFunction - identifier - (TRANSIENT)">data</span><span class="operator">(),</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span><span class="operator">);</span>
<span class="operator">}</span></pre>

With this main will now return our initial value of five. We'll now go back and do a few things, first we need to keep track of how many bytes have been written so we know where the next serialize call needs to start from and we'll need to do the same for deserializing by keeping track of how many bytes have been read. On top of that we'll also do a check to make sure we have enough bytes to store any serialization attempt and grow the vector if not (we have to do this check manually as we're operating on the underlying container of the vector itself).

<pre class="visualstudio"><span class="keyword">void</span>&nbsp;<span class="cppMemberFunction - identifier - (TRANSIENT)">serialize</span><span class="operator">(</span><span class="keyword">int</span><span class="operator">*</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">start</span><span class="operator">,</span>&nbsp;<span class="keyword">int</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span><span class="operator">)</span>
<span class="operator">{</span>
	<span class="keyword">if</span>&nbsp;<span class="operator">(</span><span class="cppMemberField - identifier - (TRANSIENT)">bytes_written</span>&nbsp;<span class="operator">+</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span>&nbsp;<span class="operator">&gt;</span>&nbsp;<span class="cppMemberField - identifier - (TRANSIENT)">bytes</span><span class="operator">.</span><span class="cppMemberFunction - identifier - (TRANSIENT)">size</span><span class="operator">())</span>
	<span class="operator">{</span>
		<span class="cppMemberField - identifier - (TRANSIENT)">bytes</span><span class="operator">.</span><span class="cppMemberFunction - identifier - (TRANSIENT)">resize</span><span class="operator">(</span><span class="cppMemberField - identifier - (TRANSIENT)">bytes_written</span>&nbsp;<span class="operator">+</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span><span class="operator">);</span>
	<span class="operator">}</span>
 
	<span class="cppFunction - identifier - (TRANSIENT)">memcpy</span><span class="operator">(</span><span class="cppMemberField - identifier - (TRANSIENT)">bytes</span><span class="operator">.</span><span class="cppMemberFunction - identifier - (TRANSIENT)">data</span><span class="operator">()</span>&nbsp;<span class="operator">+</span>&nbsp;<span class="cppMemberField - identifier - (TRANSIENT)">bytes_written</span><span class="operator">,</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">start</span><span class="operator">,</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span><span class="operator">);</span>
	<span class="cppMemberField - identifier - (TRANSIENT)">bytes_written</span>&nbsp;<span class="operator">+=</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span><span class="operator">;</span>
<span class="operator">}</span></pre>

We do a check to make sure we have enough space to serialize a value by checking  `.size()`. We could check `.capacity()` instead but writing past the size of a vector is undefined behaviour. The downside of using `.size()` is that we have to call `.resize()` each time, which default-initializes each byte to 0 (a cost which we don't need to bear). We could [define our own allocator](https://stackoverflow.com/a/12787599) that doesn't do anything so that the only side effect of calling `.resize()` is that of the vector internally knowing the size.

What we'll do instead is tightly couple resize to capacity, so we'll treat the size as the capacity and then resizing the vector will cause the vector to reallocate with extra space default initialized. This is slightly better than what we're currently doing because rather than doing default-initialization one serialize call at a time, we do it all in one big lump (as long as the next call doesn't also extend past the capacity of the vector, which is highly unlikely). If we detect any performance issues with this default-initialization we can go back and create our own allocator.

The only difference is that rather than resizing so it's just enough to serialize our data thus forcing a reallocation with an unknown size, we double the value of the new size so that the STL implementation uses that one rather than their geometric growth calculation. For instance in MSVC it's the following (excuse the formatting, I'm unsure why they would choose to make it look like this)

<pre class="visualstudio"><span class="cppType - identifier - (TRANSIENT)">size_type</span>&nbsp;<span class="cppMemberFunction - identifier - (TRANSIENT)">_Calculate_growth</span><span class="operator">(</span><span class="keyword">const</span>&nbsp;<span class="cppType - identifier - (TRANSIENT)">size_type</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">_Newsize</span><span class="operator">)</span>&nbsp;<span class="keyword">const</span>
		<span class="operator">{</span>	<span class="comment">//&nbsp;given&nbsp;_Oldcapacity&nbsp;and&nbsp;_Newsize,&nbsp;calculate&nbsp;geometric&nbsp;growth</span>
		<span class="keyword">const</span>&nbsp;<span class="cppType - identifier - (TRANSIENT)">size_type</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">_Oldcapacity</span>&nbsp;<span class="operator">=</span>&nbsp;<span class="cppMemberFunction - identifier - (TRANSIENT)">capacity</span><span class="operator">();</span>
 
		<span class="keyword">if</span>&nbsp;<span class="operator">(</span><span class="cppLocalVariable - identifier - (TRANSIENT)">_Oldcapacity</span>&nbsp;<span class="operator">&gt;</span>&nbsp;<span class="cppMemberFunction - identifier - (TRANSIENT)">max_size</span><span class="operator">()</span>&nbsp;<span class="operator">-</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">_Oldcapacity</span>&nbsp;<span class="operator">/</span>&nbsp;<span class="number">2</span><span class="operator">)</span>
			<span class="operator">{</span>
			<span class="keyword">return</span>&nbsp;<span class="operator">(</span><span class="cppParameter - identifier - (TRANSIENT)">_Newsize</span><span class="operator">);</span>	<span class="comment">//&nbsp;geometric&nbsp;growth&nbsp;would&nbsp;overflow</span>
			<span class="operator">}</span>
 
		<span class="keyword">const</span>&nbsp;<span class="cppType - identifier - (TRANSIENT)">size_type</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">_Geometric</span>&nbsp;<span class="operator">=</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">_Oldcapacity</span>&nbsp;<span class="operator">+</span>&nbsp;<span class="cppLocalVariable - identifier - (TRANSIENT)">_Oldcapacity</span>&nbsp;<span class="operator">/</span>&nbsp;<span class="number">2</span><span class="operator">;</span>
 
		<span class="keyword">if</span>&nbsp;<span class="operator">(</span><span class="cppLocalVariable - identifier - (TRANSIENT)">_Geometric</span>&nbsp;<span class="operator">&lt;</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">_Newsize</span><span class="operator">)</span>
			<span class="operator">{</span>
			<span class="keyword">return</span>&nbsp;<span class="operator">(</span><span class="cppParameter - identifier - (TRANSIENT)">_Newsize</span><span class="operator">);</span>	<span class="comment">//&nbsp;geometric&nbsp;growth&nbsp;would&nbsp;be&nbsp;insufficient</span>
			<span class="operator">}</span>
 
		<span class="keyword">return</span>&nbsp;<span class="operator">(</span><span class="cppLocalVariable - identifier - (TRANSIENT)">_Geometric</span><span class="operator">);</span>	<span class="comment">//&nbsp;geometric&nbsp;growth&nbsp;is&nbsp;sufficient</span>
		<span class="operator">}</span></pre>
		
The geometric size would be `_Oldcapacity` + 50%, so if we request `.capacity() * 2` this implementation has to use the size we gave it, ensuring `.size()` and `.capacity()` return the same value.

Of course this isn't defined behaviour either as each STL implementation can use whatever growth factor it likes, however even if for some reason an implementation grows it by more than twice the original size it won't affect the correctness of our class, it will only cause some early re-sizing as we detect that our required size is larger than the vectors size, which may no longer be the same as its capacity.

Maybe the custom allocator was a better idea after all, or even just creating our own custom container. Ah well.

<pre class="visualstudio"><span class="keyword">void</span>&nbsp;<span class="cppMemberFunction - identifier - (TRANSIENT)">serialize</span><span class="operator">(</span><span class="keyword">int</span><span class="operator">*</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">start</span><span class="operator">,</span>&nbsp;<span class="keyword">int</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span><span class="operator">)</span>
<span class="operator">{</span>
	<span class="keyword">if</span>&nbsp;<span class="operator">(</span><span class="cppMemberField - identifier - (TRANSIENT)">bytes_written</span>&nbsp;<span class="operator">+</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span>&nbsp;<span class="operator">&gt;</span>&nbsp;<span class="cppMemberField - identifier - (TRANSIENT)">bytes</span><span class="operator">.</span><span class="cppMemberFunction - identifier - (TRANSIENT)">size</span><span class="operator">())</span>
	<span class="operator">{</span>
		<span class="cppMemberField - identifier - (TRANSIENT)">bytes</span><span class="operator">.</span><span class="cppMemberFunction - identifier - (TRANSIENT)">resize</span><span class="operator">((</span><span class="cppMemberField - identifier - (TRANSIENT)">bytes_written</span>&nbsp;<span class="operator">+</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span><span class="operator">)</span>&nbsp;<span class="operator">*</span>&nbsp;<span class="number">2</span><span class="operator">);</span>
	<span class="operator">}</span>
 
	<span class="cppFunction - identifier - (TRANSIENT)">memcpy</span><span class="operator">(</span><span class="cppMemberField - identifier - (TRANSIENT)">bytes</span><span class="operator">.</span><span class="cppMemberFunction - identifier - (TRANSIENT)">data</span><span class="operator">()</span>&nbsp;<span class="operator">+</span>&nbsp;<span class="cppMemberField - identifier - (TRANSIENT)">bytes_written</span><span class="operator">,</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">start</span><span class="operator">,</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span><span class="operator">);</span>
	<span class="cppMemberField - identifier - (TRANSIENT)">bytes_written</span>&nbsp;<span class="operator">+=</span>&nbsp;<span class="cppParameter - identifier - (TRANSIENT)">length</span><span class="operator">;</span>
<span class="operator">}</span></pre>

Now we request `(bytes_written + length) * 2` which is actually more than `.capacity() * 2`, so size and capacity should be equal (most of the time);

We don't want to limit ourselves to serializing and deserializing ints. We could make some templated functions as all we're operating on is pointers and sizes however then they'd work for any type (including `MyCoolClassWithPointers`) and we don't want that. We could use `if constexpr(std::is_integral<T>::value)` which would limit us to basic types and then provide template specialication overloads for types such as `std::string` and that would be perfectly okay a lot of the time. This packet class is meant to be used to send data over the network though so we need to care about Network Byte Order and Host Byte Order and converting from one to the other (for instance with `ntohl()`).

We can't simply flip the endianess of the entire byte container because that would affect the order in which we read from the container on the other machine. For instance if we have (a, b) `0x00112233` and flip it to (b, a) `0x33221100`, then the other machine reads it in and keeps it like that (because that would be its host endianess) and then tries to deserialize it the deserialization would begin from (b) `0x33` rather than (a) `0x00` producing incorrect output. If instead we do it on a per-type basis then (a, b) `0x00112233` becomes (a, b) `0x11003322` which gets sent across the networking and then deserialized in the correct order (a would be `0x1100` and b would be `0x3322`, which because of the different endianess is correct).

As we're going to have to convert the endianess of these different types there's no value in templating this, instead we'll have to create overloads for every type we want to serialize and deserialize. Dealing with endianess is an issue to explore in another post, so for now we'll ignore it.

- Keeping track of size bytes seperately because when we reset we also want to reset the size, but then that causes issues because blahblahblah
- add p.read<var args>() so you can do f(p.read<Args>()...)
- add reset
- packet class that is read-only (is const enough?) packet_view?
- packet header
- combining packet buffers together
- string and string_view overload
- replace memcpy with reinterpret cast? performance?


