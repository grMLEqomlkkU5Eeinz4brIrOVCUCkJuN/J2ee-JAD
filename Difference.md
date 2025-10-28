# JSP vs EJS

## Basic Syntax Comparison

| Feature | JSP | EJS |
|---------|-----|-----|
| **Scriptlet** | `<% code %>` | `<% code %>` |
| **Expression** | `<%= expression %>` | `<%= expression %>` |
| **Comment** | `<%-- comment --%>` | `<%# comment %>` |
| **Declaration** | `<%! declaration %>` | N/A (use `<% var x; %>`) |
| **HTML Comment** | `<!-- comment -->` | `<!-- comment -->` |

## Directives vs Configuration

### JSP Directives
```jsp
<%@ page language="java" contentType="text/html" %>
<%@ include file="header.jsp" %>
<%@ taglib uri="tagLibraryURI" prefix="prefix" %>
```

### EJS Configuration
```javascript
// Set in Node.js/Express
app.set('view engine', 'ejs');
app.set('views', './views');
```

## Including Files

### JSP
```jsp
<!-- Static include (compile-time) -->
<%@ include file="header.jsp" %>

<!-- Dynamic include (runtime) -->
<jsp:include page="header.jsp" flush="true"/>
<jsp:include page="header.jsp">
    <jsp:param name="title" value="Home"/>
</jsp:include>
```

### EJS
```ejs
<!-- Include partial -->
<%- include('header') %>

<!-- Include with data -->
<%- include('header', {title: 'Home'}) %>
```

## Control Flow

### JSP
```jsp
<% if (user != null) { %>
    <p>Welcome, <%= user.getName() %></p>
<% } else { %>
    <p>Please log in</p>
<% } %>

<% for (int i = 0; i < items.length; i++) { %>
    <li><%= items[i] %></li>
<% } %>
```

### EJS
```ejs
<% if (user) { %>
    <p>Welcome, <%= user.name %></p>
<% } else { %>
    <p>Please log in</p>
<% } %>

<% items.forEach(item => { %>
    <li><%= item %></li>
<% }); %>
```

## Working with Beans/Objects

### JSP Beans
```jsp
<jsp:useBean id="user" class="com.example.User" scope="session"/>
<jsp:setProperty name="user" property="name" value="John"/>
<jsp:getProperty name="user" property="name"/>
```

### EJS Objects
```ejs
<!-- Data passed from Express route -->
<%= user.name %>
<%= user.email %>
```

## Request/Response Handling

### JSP Implicit Objects
```jsp
<%= request.getParameter("username") %>
<%= session.getAttribute("userId") %>
<%= application.getInitParameter("appName") %>
<% response.setContentType("text/html"); %>
<% out.println("Hello World"); %>
```

### EJS with Express
```javascript
// In Express route
app.get('/user', (req, res) => {
    res.render('user', {
        username: req.query.username,
        userId: req.session.userId,
        appName: process.env.APP_NAME
    });
});
```

## Error Handling

### JSP
```jsp
<%@ page errorPage="error.jsp" %>
<%@ page isErrorPage="true" %>
<%= exception.getMessage() %>
```

### EJS
```javascript
// Express error middleware
app.use((err, req, res, next) => {
    res.render('error', { error: err });
});
```

## Page Navigation

### JSP Forward
```jsp
<jsp:forward page="welcome.jsp">
    <jsp:param name="user" value="John"/>
</jsp:forward>
```

### EJS/Express Redirect
```javascript
res.redirect('/welcome?user=John');
```

## Output Escaping

| Type | JSP | EJS |
|------|-----|-----|
| **Escaped** | `<%= expression %>` | `<%= expression %>` |
| **Unescaped** | `<%! expression %>` (different purpose) | `<%- expression %>` |
