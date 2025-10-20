-----

# 🟢 oLoader: Dynamic Content Assembler

**oLoader** is a lightweight, zero-dependency JavaScript module designed to manage client-side content assembly, dependency ordering, and data hydration in a static, runtime environment. It delivers **modern application structure without modern application overhead**. It’s perfect for adding dynamic, component-based power to simple HTML files without needing a complex build step.

-----

## 💡 Features

  * **Runtime Assembly:** Asynchronously fetches and inserts HTML, CSS, and Scripts into the `<head>` or `<body>`.
  * **Guaranteed Script Order:** Scripts are executed sequentially, ensuring dependencies load correctly.
  * **Dynamic UI Management:** The **`element.put()`** method enables **view swapping** and seamless content replacement *after* initial assembly.
  * **Imperative Form Handling:** The **`element.ifsubmit()`** method simplifies form submission by preventing default behavior and parsing data into a clean object.
  * **Clean Element Selection:** The global **`oG()`** utility simplifies DOM querying, making code cleaner and immediately readable.
  * **Global Data Hydration:** The integrated `window.paint` utility provides fast, attribute-based templating.
  * **Simple Lifecycle:** Uses a single `.load(callback)` method for clean initialization and cleanup.

-----

## 🚀 Getting Started

### 1\. Installation

Download the latest `oloader.js` and include it, followed by your application logic, in your main HTML file. **oLoader** is ready immediately on page load.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>oLoader Application</title>
    <script src="path/to/oloader.js"></script>
    
    <script src="path/to/app.js"></script>
</head>
<body>
    <div id="app-container">
        <p>Loading application structure...</p>
    </div>
</body>
</html>
```

### 2\. Basic Usage and The `oG()` Selector

Use the `oLoader()` constructor to create an assembly instance, queue your files, and call `.load()` to execute the process. Use the **`oG()`** utility to simplify element selection and immediately apply logic.

```javascript
// app.js

// 1. Initialize the assembly core
const app = oLoader();

// 2. Queue content and dependencies
app.head('/assets/styles.css', 'e');          // Load styles
app.body('/components/main-view.html', 'e');  // Load component structure
app.script('/app/router.js');                 // Load logic

// 3. Execute assembly and run final initialization
app.load(() => {
    console.log('Application structure assembled and ready!');
    
    // ⭐ Use oG() to easily select elements and apply logic ⭐
    
    // Example 1: Attach an ifsubmit handler to the login form
    oG('#login-form', (form) => {
        // 'form' is the HTMLElement object
        form.ifsubmit(({ data }) => {
            console.log('Login attempt:', data);
        });
    });

    // Example 2: Load a dynamic component into a specific container
    oG('#dashboard-container').put('/views/user-welcome.html', null, () => {
        console.log('Welcome message loaded!');
    });
});
```

-----

## 🎨 Data Hydration (`window.paint`)

The global `paint` utility is used to inject data into loaded HTML templates. It searches for elements marked with the **`.use-painter`** class and the **`painter-data`** attribute.

### Templating

`paint` automatically targets attributes like `src` (for `<img>`) and `href` (for `<a>`). Otherwise, it updates `textContent`.

```html
<div class="card">
    <h3 class="use-painter" painter-data="userName"></h3>
    
    <img class="use-painter" painter-data="avatarURL" src="" alt="Avatar">
    
    <a class="use-painter" painter-data="profileID" href="/user/">View</a>
</div>
```

### Hydrating the Data

You typically call `paint.obj()` inside the `.load()` callback once data is available.

```javascript
// 1. Start fetching data concurrently
const userData = fetch('/api/user/current').then(res => res.json());

app.load(() => {
    userData.then(data => {
        // Hydrate the structure with data
        paint.obj({
            userName: data.name,
            avatarURL: data.image_url,
            profileID: data.id // The <a> href becomes /user/123
        });
    });
});
```

-----

## 📚 Core API Reference

The API is divided into methods for initial assembly and methods available on all elements for dynamic runtime interactions.

### Initial Assembly Methods (The `app` Instance)

| Method | Arguments | Description |
| :--- | :--- | :--- |
| **`oLoader()`** | *None* | Initializes and returns the assembly instance (`app`). |
| **`app.head(file, pos)`** | `string`, `string` ('b' or 'e') | Queues content for insertion into `<head>`. |
| **`app.body(file, pos)`** | `string`, `string` ('b' or 'e') | Queues content for insertion into `<body>`. |
| **`app.script(file)`** | `string` | Queues a JS file for **sequential execution**. |
| **`app.load(callback)`** | `function` | Triggers fetching and runs the `callback` when **all** assets are ready. |

\<hr\>

### Dynamic Element Methods (Available **After** `app.load()`)

These methods are available on **any HTML Element** for managing runtime content updates and user interactions.

#### 1\. Dynamic Content: `element.put(file, [position], [callback])`

Asynchronously fetches content and inserts it into the target element. Returns a `Promise` and accepts an optional callback.

| Position Value | Action | Clears Existing Content? | Use Case |
| :--- | :--- | :--- | :--- |
| **(Omitted or `null`)** | Replaces the element's **entire content**. | **YES** | **Seamless State Transition** (e.g., replacing a loading spinner). |
| **`e`** (End/Clear) | Appends content to the end. | **YES** | **View Swapping** (replacing the old view with a new one). |
| **`E`** (End/No Clear) | Appends content to the end. | No | **Infinite Scroll/Pagination** (adding more items). |
| **`B`** (Begin/No Clear) | Prepends content to the beginning. | No | Inserting a new item at the top (e.g., new chat message). |

#### 2\. Form Handling: `element.ifsubmit(callback)`

Attaches a submit handler to the element (intended for a `<form>`). It **prevents the default browser submission** and collects all form data into a structured object for easy processing.

| Callback Argument Key | Type | Description |
| :--- | :--- | :--- |
| **`caller`** | `HTMLElement` | The element (button) that initiated the submission. |
| **`form`** | `HTMLElement` | The form element itself. |
| **`location`** | `string` | The value of the form's `action` attribute. |
| **`data`** | `Object` | Key-value pairs of all form fields (e.g., `{ inputName: value }`). |

\<hr\>

### Global Utilities

| Method | Arguments | Description |
| :--- | :--- | :--- |
| **`oG(selector, [callback])`** | `string`, `function` | **The Element Selector.** Retrieves element(s) based on a CSS selector. Returns a single `HTMLElement` for IDs or an `Array` for classes/tags. If a `callback` is provided, it executes the callback immediately on the element(s). |
| **`paint.obj(data, [element])`** | `Object`, `HTMLElement` | Hydrates the DOM using a single object. If `element` is provided, it scopes the search for `.use-painter` to that element. |
| **`paint.array(container, array)`** | `HTMLElement`, `Array` | Renders a list by cloning the container's first child for each item in the array. |

-----

## 🤝 Contributing

We welcome contributions\! Please read our [CONTRIBUTING.md](https://www.google.com/search?q=CONTRIBUTING.md) for guidelines on submitting pull requests, reporting bugs, and suggesting features.

-----

## 📜 License

oLoader is [Licensed under the MIT License](https://www.google.com/search?q=LICENSE).
