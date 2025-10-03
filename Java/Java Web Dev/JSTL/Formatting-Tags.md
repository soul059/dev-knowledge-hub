# JSTL Formatting Tags

JSTL (JavaServer Pages Standard Tag Library) formatting tags are used to format and display text, numbers, and dates in JSP pages for internationalization and localization. To use the formatting tags, you must include the following directive in your JSP file:

```jsp
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
```

Here are some of the most common JSTL formatting tags:

### Number Formatting

The `<fmt:formatNumber>` tag is used to format numeric values as numbers, currencies, or percentages.

**Attributes:**

*   `value`: The numeric value to be formatted.
*   `type`: Can be `number`, `currency`, or `percent`.
*   `pattern`: A custom pattern for formatting.
*   `currencyCode`: The ISO 4217 currency code (e.g., "USD").
*   `currencySymbol`: The currency symbol (e.g., "$").
*   `groupingUsed`: A boolean to specify whether to group numbers with commas.

**Example:**

```jsp
<c:set var="balance" value="123456.789" />

<p>Formatted Number: <fmt:formatNumber value="${balance}" type="number" /></p>
<p>Currency: <fmt:formatNumber value="${balance}" type="currency" currencySymbol="$" /></p>
<p>Percentage: <fmt:formatNumber value="0.75" type="percent" /></p>
<p>Custom Pattern: <fmt:formatNumber value="${balance}" pattern="#,##0.00" /></p>
```

### Date and Time Formatting

The `<fmt:formatDate>` tag is used to format dates and times.

**Attributes:**

*   `value`: The date/time value to be formatted.
*   `type`: Can be `date`, `time`, or `both`.
*   `dateStyle`: Predefined style for dates (`default`, `short`, `medium`, `long`, `full`).
*   `timeStyle`: Predefined style for times (`default`, `short`, `medium`, `long`, `full`).
*   `pattern`: A custom pattern for formatting (e.g., "yyyy-MM-dd HH:mm:ss").

**Example:**

```jsp
<jsp:useBean id="now" class="java.util.Date" />

<p>Default Date: <fmt:formatDate value="${now}" /></p>
<p>Short Date: <fmt:formatDate value="${now}" dateStyle="short" /></p>
<p>Long Time: <fmt:formatDate value="${now}" type="time" timeStyle="long" /></p>
<p>Both (Full): <fmt:formatDate value="${now}" type="both" dateStyle="full" timeStyle="full" /></p>
<p>Custom Pattern: <fmt:formatDate value="${now}" pattern="MM/dd/yyyy" /></p>
```

### Internationalization (I18N)

Formatting tags are locale-sensitive and are a key part of creating internationalized web applications.

*   `<fmt:setLocale>`: Sets the locale for a given scope.
*   `<fmt:bundle>` and `<fmt:message>`: Used to display localized messages from resource bundles.

**Example:**

```jsp
<fmt:setLocale value="fr_FR"/>
<p>French Currency: <fmt:formatNumber value="123456.789" type="currency" /></p>
<p>French Date: <fmt:formatDate value="${now}" dateStyle="full" /></p>
```

This would format the number and date according to French conventions.

### Number and Date Parsing

The `<fmt:parseNumber>` and `<fmt:parseDate>` tags are used to parse strings into numbers or dates, respectively. This is useful when handling user input.

**`<fmt:parseNumber>` Attributes:**

*   `value`: The string to be parsed.
*   `type`: The type of number (`number`, `currency`, or `percent`).
*   `pattern`: A custom pattern for parsing.
*   `var`: The name of the variable to store the parsed number.
*   `scope`: The scope of the variable.

**`<fmt:parseDate>` Attributes:**

*   `value`: The string to be parsed.
*   `type`: The type of date (`date`, `time`, or `both`).
*   `pattern`: A custom pattern for parsing.
*   `var`: The name of the variable to store the parsed date.
*   `scope`: The scope of the variable.

**Parsing Example:**

```jsp
<%-- Parsing a number string --%>
<c:set var="numberString" value="1,234.56" />
<fmt:parseNumber var="parsedNumber" value="${numberString}" type="number" />
<p>Parsed Number: ${parsedNumber}</p>

<%-- Parsing a date string --%>
<c:set var="dateString" value="2023-12-25" />
<fmt:parseDate var="parsedDate" value="${dateString}" pattern="yyyy-MM-dd" />
<p>Parsed Date: <fmt:formatDate value="${parsedDate}" dateStyle="full" /></p>
```
