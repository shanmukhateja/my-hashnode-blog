---
title: "How to export HTML table to PDF in Angular"
datePublished: Mon Nov 13 2023 10:39:16 GMT+0000 (Coordinated Universal Time)
cuid: clowrvg8z000t09im4o0i2sne
slug: how-to-export-html-table-to-pdf-in-angular
tags: javascript, angularjs, canvas, html5, pdf

---

Hello,

In this post, I will explain how to convert an HTML table to PDF. This applies to other HTML widgets but I will be focusing on `table` which has a large dataset and vertical scrolling to not clutter the page.

### Prerequisites

We are going to use 2 libraries to achieve this.

1. [html2canvas](https://github.com/niklasvh/html2canvas)
    
2. [jspdf](https://www.npmjs.com/package/jspdf)
    

> I am using the following versions for this example taken from my package.json.
> 
> ```json
> {
>   "html2canvas": "^1.4.1",
>   "jspdf": "^2.5.1"
> }
> ```

### Disclaimer

#### **html2canvas:**

As the name suggests, `html2canvas` processes and converts a given HTML DOM node into an image using `canvas` under the hood while keeping the styling intact.

1. This library is considered "experimental" (as of writing) by its author and should not be used for production.
    
2. Please take a look at the [Features list](https://html2canvas.hertzen.com/features) and verify whether this library supports all features that are required by your web app.
    

#### jsPDF:

It is used to generate a PDF file based on the image (created from the `html2canvas` library).

Learn more on their GitHub [here](https://github.com/parallax/jsPDF).

### Theory

The idea is very simple:

1. We use html2canvas to generate an `canvas` object with the table.
    
2. We then use the `canvas#toDataURL` function ([MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toDataURL)) to generate a data URL.
    
3. We initialize `jsPDF`, use `addImage` function to set the image via data URL.
    
4. ..
    
5. Profit?
    

### Implementation

1. Let's create a new Angular project.
    

```bash
$ ng new testpdf
```

1. Install both the libraries from NPM
    

```bash
$ npm install jspdf html2canvas
```

1. HTML code for Angular component
    

```xml
<!-- app.component.html -->
  
  <!-- Export button -->
  <div class="my-3">
    <button class="btn btn-primary" (click)="export()">Export to PDF</button>
  </div>
  
  <!-- Table -->
  <h3>Preview:</h3>
  <div class="table-container">
  
    <table id="data-table" class="table table-bordered">
      <thead>
        <tr>
          <th>ID</th>
          <th>First name</th>
          <th>Last name</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>3</td>
          <td>Cartman</td>
          <td>Whateveryournameis</td>
        </tr>
        <tr>
          <td>10</td>
          <td>Cartman</td>
          <td>Titi</td>
        </tr>
        <tr>
          <td>11</td>
          <td>Toto</td>
          <td>Lara</td>
        </tr>
        <tr>
          <td>22</td>
          <td>Luke</td>
          <td>Yoda</td>
        </tr>
        <tr>
          <td>26</td>
          <td>Foo</td>
          <td>Moliku</td>
        </tr>
        <tr>
          <td>31</td>
          <td>Luke</td>
          <td>Someone Last Name</td>
        </tr>
        <tr>
          <td>32</td>
          <td>Batman</td>
          <td>Lara</td>
        </tr>
        <tr>
          <td>37</td>
          <td>Zed</td>
          <td>Kyle</td>
        </tr>
        <tr>
          <td>39</td>
          <td>Louis</td>
          <td>Whateveryournameis</td>
        </tr>
        <tr>
          <td>41</td>
          <td>Superman</td>
          <td>Yoda</td>
        </tr>
        <tr>
          <td>42</td>
          <td>Batman</td>
          <td>Moliku</td>
        </tr>
        <tr>
          <td>43</td>
          <td>Zed</td>
          <td>Lara</td>
        </tr>
        <tr>
          <td>46</td>
          <td>Foo</td>
          <td>Someone Last Name</td>
        </tr>
        <tr>
          <td>47</td>
          <td>Superman</td>
          <td>Someone Last Name</td>
        </tr>
        <tr>
          <td>48</td>
          <td>Toto</td>
          <td>Bar</td>
        </tr>
        <tr>
          <td>48</td>
          <td>Batman</td>
          <td>Lara</td>
        </tr>
        <tr>
          <td>54</td>
          <td>Luke</td>
          <td>Bar</td>
        </tr>
        <tr>
          <td>62</td>
          <td>Foo</td>
          <td>Kyle</td>
        </tr>
        <tr>
          <td>80</td>
          <td>Zed</td>
          <td>Kyle</td>
        </tr>
        <tr>
          <td>87</td>
          <td>Zed</td>
          <td>Someone Last Name</td>
        </tr>
        <tr>
          <td>87</td>
          <td>Toto</td>
          <td>Yoda</td>
        </tr>
        <tr>
          <td>88</td>
          <td>Toto</td>
          <td>Titi</td>
        </tr>
        <tr>
          <td>89</td>
          <td>Luke</td>
          <td>Whateveryournameis</td>
        </tr>
        <tr>
          <td>97</td>
          <td>Zed</td>
          <td>Bar</td>
        </tr>
        <tr>
          <td>101</td>
          <td>Someone First Name</td>
          <td>Someone Last Name</td>
        </tr>
        <tr>
          <td>104</td>
          <td>Toto</td>
          <td>Kyle</td>
        </tr>
        <tr>
          <td>105</td>
          <td>Toto</td>
          <td>Titi</td>
        </tr>
        <tr>
          <td>107</td>
          <td>Cartman</td>
          <td>Whateveryournameis</td>
        </tr>
        <tr>
          <td>107</td>
          <td>Louis</td>
          <td>Lara</td>
        </tr>
        <tr>
          <td>113</td>
          <td>Foo</td>
          <td>Moliku</td>
        </tr>
        <tr>
          <td>114</td>
          <td>Someone First Name</td>
          <td>Titi</td>
        </tr>
        <tr>
          <td>119</td>
          <td>Zed</td>
          <td>Someone Last Name</td>
        </tr>
        <tr>
          <td>121</td>
          <td>Toto</td>
          <td>Bar</td>
        </tr>
        <tr>
          <td>131</td>
          <td>Louis</td>
          <td>Moliku</td>
        </tr>
        <tr>
          <td>133</td>
          <td>Cartman</td>
          <td>Moliku</td>
        </tr>
        <tr>
          <td>134</td>
          <td>Someone First Name</td>
          <td>Someone Last Name</td>
        </tr>
        <tr>
          <td>134</td>
          <td>Toto</td>
          <td>Whateveryournameis</td>
        </tr>
        <tr>
          <td>135</td>
          <td>Superman</td>
          <td>Whateveryournameis</td>
        </tr>
        <tr>
          <td>144</td>
          <td>Someone First Name</td>
          <td>Yoda</td>
        </tr>
        <tr>
          <td>154</td>
          <td>Luke</td>
          <td>Moliku</td>
        </tr>
        <tr>
          <td>154</td>
          <td>Batman</td>
          <td>Bar</td>
        </tr>
        <tr>
          <td>155</td>
          <td>Louis</td>
          <td>Whateveryournameis</td>
        </tr>
        <tr>
          <td>156</td>
          <td>Someone First Name</td>
          <td>Lara</td>
        </tr>
      </tbody>
    </table>
  </div>
```

1. CSS styling (optional)
    

```css
/* app.component.css */
.table-container {
  height: 50vh;
  overflow: auto;
}
```

1. Attempt - 1:  
    `exportPDF` function
    

```typescript
// app.component.ts
export() {
    // 1. Get a reference to the DOM node.
    const component = document.getElementById('data-table')!;
    // 2. Get a reference to width, height of DOM node.
    const componentWidth = component.offsetWidth
    const componentHeight = component.offsetHeight

    // 3. Generate <canvas> from HTML node.
    html2canvas(component).then((canvas) => {
      // 4. Generate Data URL from <canvas>
      const imgData = canvas.toDataURL('image/png');
      // 5. Create an instance of `jsPDF`
        // 5.1 `orientation`  -> 'landscape' || 'portrait'
        // 5.2 `unit`         -> 'px' (mandatory)
      const pdf = new jsPDF({ orientation: 'landscape', unit: 'px'});
      // 6. Use `addImage` to render generated image into PDF page.
      pdf.addImage(imgData, 'PNG', 0, 0, componentWidth, componentHeight)
      // 7. Download PDF
      pdf.save('filename.pdf')
    })
  }
```

### Results - I

#### Expected output:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699869769360/be9a111d-80b0-4885-b775-0cc99f78a015.png align="center")

#### Actual output:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699870070013/8419f72e-22b4-44d5-9eb8-1734676dd8c3.png align="left")

So, what went wrong? There are 3 observations to be made.

1. The contents inside the PDF file are distorted.
    
2. The content hasn't scaled very well making it blurry (I mean look at the actual result screenshot!).
    
3. The third column is entirely missing.
    

After researching this for a while, I came across [**this**](https://stackoverflow.com/questions/36472094/how-to-set-image-to-fit-width-of-the-page-using-jspdf/64761137#64761137) StackOverflow post which helped me understand the issue.

My theory is that the width and height properties of the jsPDF's internal page are much smaller than the `<table>` DOM node. This means we are trying to fit something so big inside something much smaller. So, when we try to fit in a "large" image inside a jsPDF page, it would (obviously) be cut off.

So, how do we fix this?

We need to somehow adjust the width and height properties of the PDF file to match the DOM's width and height.

Let's try this again.

```typescript
// app.component.ts
export() {
    // 1. Get a reference to the DOM node.
    const component = document.getElementById('data-table')!;
    // 2. Get a reference to width, height of DOM node.
    const componentWidth = component.offsetWidth
    const componentHeight = component.offsetHeight

    // 3. Generate <canvas> from HTML node.
    html2canvas(component).then((canvas) => {
      // 4. Generate Data URL from <canvas>
      const imgData = canvas.toDataURL('image/png');
      // 5. Create an instance of `jsPDF`
        // 5.1 `orientation`  -> 'landscape' || 'portrait'
        // 5.2 `unit`         -> 'px' (mandatory)
      const pdf = new jsPDF({ orientation: 'landscape', unit: 'px'});

      // NEW - set jsPDF page width/height the same as the DOM node.
      pdf.internal.pageSize.width = componentWidth;
      pdf.internal.pageSize.height = componentHeight;
   
      // 6. Use `addImage` to render generated image into PDF page.
      pdf.addImage(imgData, 'PNG', 0, 0, componentWidth, componentHeight)
      // 7. Download PDF
      pdf.save('filename.pdf')
    })
  }
```

### Results - II

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699870834860/6ee902a8-3b3c-4ad1-8cbb-73334ecba46f.png align="center")

Yup! It works.

> If you're wondering whether the "Actual Results" image from above was taken from the output of this code, then you're right :P

### Bonus content

There is a chance you're seeing large file sizes on the generated PDF files. This could happen if your table uses too much CSS.  
The solution is to enable `compress: true` option in `jsPDF` initialization options like so:

```typescript
// app.component.ts
export() {
    ...

    const pdf = new jsPDF({
        ...,
        compress: true    // reduces PDF file size
    });

    ...
 }
```

### Conclusion

I hope you've learned something in this post.

If you've liked this post, show your support by using the emojis on the right. It pleases the algorithm :^)

You can also @ me on Mastodon [here](http://social.linux.pizza/@shanmukhateja).

Bye for now.