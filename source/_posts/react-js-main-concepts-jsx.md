---
title: 筆記 - ReactJS 主要概念 - JSX
tags:
  - ReactJS
  - JSX
date: 2022-07-30 10:49:00
---

## 概念
顧名思義，JSX 就是 JavaScript XML 的縮寫，是一種 ，讓開發的人可以直接把 HTML 的標籤寫在 JavaScript 裡面。這樣一來，可以省去操作原生 DOM API 的麻煩，讓產生 HTML 的方式變簡單。這種語法被稱作是**語法糖**(Syntax Sugar)。

## JSX 與樣板語言的比較
和其他樣板語言(例如：[Handlebars](https://handlebarsjs.com/), [Mustache](https://github.com/janl/mustache.js)) 不一樣的地方，就是可以直接在其中撰寫 JavaScript 程式碼。

樣板語言一般會要求使用到它們自身的語法，才能顯示變數或是執行運算式；而且，通常是分開撰寫，由 JavaScript 定義規範好的資料結構並傳入樣板中，再以樣板語法按照規範顯示。以下是 [Handlebars](https://handlebarsjs.com/) 的範例：

``` html
<!-- user.html -->
<p>{{person.firstname}} {{person.lastname}}</p>
```

``` javascript
// step 1: load the html file synchronizedly
const source = load('user.html');

// step 2: initialize the template function
const template = Handlebars.compile(source);

// step 3: define the data with correct structure and pass it to template function to get the result
const data = {
  person: {
    firstname: "Yehuda",
    lastname: "Katz"
  }
};
const result = template(data);
```

如果採用 JSX 的話，只需要在 JavaScript 裡面，一次把事情做完。下面是 TSX(TypeScript 版本的 JSX) 的範例：

``` tsx
import React from 'react';

class User extends React.Component<
  { firstname: string; lastname: string }
> {
  constructor(props) {
    super(props);
  }

  render() {
    const { firstname, lastname } = this.props;
    return <p>{firstname} {lastname}</p>;
  }
}
```

## JSX 的特性
1. 當作 HTML 撰寫，但是額外需要注意的地方
``` tsx
import React from 'react';
import ReactDOM from "react-dom";

class User extends React.Component {
  constructor(props) {
    super(props);
  }

  variables = [
    { key: 1, description: 'ReactElement', content: <p>ReactElement</p> }, // ✅ 顯示: <p>ReactElement</p>
    { key: 2, description: 'ReactElement', content: <p tabIndex="1">tabIndex and tabindex</p> }, // ✅ 顯示: <p tabindex="1">tabIndex and tabindex</p>
    { key: 3, description: 'ReactElement', content: <p className="container">className and class</p> }, // ✅ 顯示: <p class="container">className and class</p>
    { key: 4, description: 'string', content: '字串' }, // ✅ 顯示: 字串
    { key: 5, description: 'number', content: 0 }, // ✅ 顯示: 0
    { key: 6, description: 'ReactFragment', content: <React.Fragment>React.Fragment</React.Fragment> }, // ✅ 顯示: React.Fragment
    { key: 7, description: 'ReactPortal', content: ReactDOM.createPortal(<>Portal</>, document.body)}, // ✅ 顯示: Portal ，但會被加在 document.body 的最後面
    { key: 8, description: 'boolean', content: true }, // ❌ 不會被顯示
    { key: 9, description: 'null', content: null }, // ❌ 不會被顯示
    { key: 10, description: 'undefined', content: undefined } // ❌ 不會被顯示
  ];

  render() {
    return (
      /*
       * 通常是一個會被繪製出來的標籤，但也可以是不被繪製出來的標籤，例如：<React.Fragment> 或是 <>。
       */
      <ul>
        {
          this.variables.map(({ /* 可以直接用 map 顯示一個清單 */
            key,
            description,
            content
          }) => (
            <li key={key}> {/* 顯示清單的項目，必須使用 key 作為控制顯示與否的條件 */}
              <label>{description}</label>: {content}
            </li>
          ))
        }
      </ul>
    );
  }
}
```
