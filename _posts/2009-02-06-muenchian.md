---
layout: post
authors: [bart_blommaerts]
title: "Muenchian Method"
tags: [Muenchian Method]
category: XLST
comments: true
---

Last week I encountered a grouping problem when designing an XLST stylesheet. The application I'm working on, receives an input XML file, that has to be parsed and converted into a PDF document using <a href="http://xmlgraphics.apache.org/fop/" target="_blank">Apache's XSLFOP</a>. The data in the XML file is structured in a way that is different from the structure of the PDF document. For example, the XML I'm receiving looks like:
<pre>&lt;transport&gt;
     &lt;cars&gt;
        &lt;brand&gt;Audi&lt;/brand&gt;
        &lt;model&gt;A4&lt;/model&gt;
        &lt;hp&gt;200&lt;/hp&gt;
     &lt;/cars&gt;
     &lt;cars&gt;
        &lt;brand&gt;BMW&lt;/brand&gt;
        &lt;model&gt;3 series&lt;/model&gt;
        &lt;hp&gt;180&lt;/hp&gt;
     &lt;/cars&gt;
...
&lt;/transport&gt;</pre>
The result in the PDF had to be sorted by brand in order to have a list with all models BMW makes. This means I had to group all the models for all the different brands. Identifying the different models is possible with <a href="http://www.w3schools.com/xpath/xpath_axes.asp" target="_blank">preceding-siblings</a>, however this method requires a lot of XML processing. Since I had no idea how many cars the input XML will contain, I needed a different solution.

I found that solution in the Muenchian Method, originally developed by <a href="http://www.oreillynet.com/pub/au/609" target="_blank">Steve Muench</a>. The Muenchian Method uses <a href="http://www.w3schools.com/xsl/el_key.asp" target="_blank">keys</a> to give you access to the different nodes (brands) through the key value. Since we want to group the cars by their brand, we create the following key:
<pre>&lt;xsl:key name="cars-by-brand" match="cars" use="brand" /&gt;</pre>
In order to obtain all models for a certain brand, we select it from the key:
<pre>&lt;xsl:apply-templates select="key('cars-by-brand',brand)" /&gt;</pre>
Our goal was to make a list of all models grouped for the brands. So every time we find a new brand, we need to start a new list.  In order to find out if a selected brand occurs the first time in the XML file, we use the Muenchian Method:
<pre>cars[count(. | key('cars-by-brand', brand)[1]) = 1]</pre>
Finally the following XSLT template returns a (sorted) list of models grouped by brand:
<pre>&lt;xsl:key name="cars-by-brand" match="cars" use="brand" /&gt;
&lt;xsl:template match="transport"&gt;
	&lt;xsl:for-each select="cars[count(. | key('cars-by-brand', brand)[1]) = 1]"&gt;
		&lt;xsl:sort select="brand" /&gt;
		&lt;xsl:value-of select="brand" /&gt;
		&lt;xsl:for-each select="key('cars-by-brand', brand)"&gt;
			&lt;xsl:sort select="model" /&gt;
			&lt;xsl:value-of select="model" /&gt;
		&lt;/xsl:for-each&gt;
	&lt;/xsl:for-each&gt;
&lt;/xsl:template&gt;</pre>
For additional information on XSLT grouping, I recommend <a href="http://www.jenitennison.com/xslt/grouping/" target="_blank">Jeni's XSLT Pages</a> on grouping.