location:dokuwiki/regex/java查找url的正则
title: java查找url的正则 
author:xbynet
modifyAt:2016-12-13 03:14:52
createAt:2016-12-13 03:14:52

#  Java查找url的正则例子 
``` java

public class LinkMarkupPlugin implements WeblogEntryCommentPlugin {
    private static final Log LOG = LogFactory.getLog(LinkMarkupPlugin.class);

    private static final Pattern PATTERN = Pattern.compile(
            "http[s]?://[^/][\\S]+", Pattern.CASE_INSENSITIVE);  
    
    public LinkMarkupPlugin() {
        LOG.debug("Instantiating LinkMarkupPlugin");
    }
    
    /**
     * Unique identifier.  This should never change. 
     */
    public String getId() {
        return "LinkMarkup";
    }
    
    
    /** Returns the display name of this Plugin. */
    public String getName() {
        return "Link Markup";
    }
    
    
    /** Briefly describes the function of the Plugin.  May contain HTML. */
    public String getDescription() {
        return "Automatically creates hyperlinks out of embedded URLs.";
    }
    
    
    /**
     * Apply plugin to the specified text.
     *
     * @param comment The comment being rendered.
     * @param text String to which plugin should be applied.
     *
     * @return Results of applying plugin to string.
     */
    public String render(final WeblogEntryComment comment, String text) {
        StringBuilder result;
        result = new StringBuilder();
        
        if (text != null) {
            Matcher matcher;
            matcher = PATTERN.matcher(text);

            int start = 0;
            int end = text.length();
            
            while (start < end) {
                if (matcher.find()) {
                    // Copy up to the match
                    result.append(text.substring(start, (matcher.start())));

                    // Copy the URL and create the hyperlink
                    // Unescape HTML as we don't know if that setting is on
                    String url;
                    url = Utilities.unescapeHTML(text.substring(
                            matcher.start(), matcher.end()));

                    // Build the anchor tag and escape HTML in the URL output
                    result.append("<a href=\"");
                    result.append(Utilities.escapeHTML(url));
                    result.append("\">");
                    result.append(Utilities.escapeHTML(url));
                    result.append("</a>");

                    // Increment the starting index
                    start = matcher.end();
                }
                else {
                    // Copy the remainder
                    result.append(text.substring(start, end));

                    // Increment the starting index to exit the loop
                    start = end;
                }
            }
        }

        return result.toString();
    }
    
}


```