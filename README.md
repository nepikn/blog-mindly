# Blog Mindly

使用 React Router 管理路由、Material UI 作為元件庫

- [預期功能](#預期功能)
- [展示](#展示)
- [API](#api)
- [主要技術](#主要技術)
- [指令](#指令)
- [學習內容](#學習內容)
- [展望](#展望)
- [相關資料](#相關資料)
- [素材](#素材)

## 預期功能

用戶可以

- 登入並瀏覽不同文章類別
- 透過上／下一頁在不同的文章類別之間切換
- 對文章表示「🙂」、「👍」

## 展示

> [部署於個人網域](https://blog.unconscious.cc)

| 首頁                                                                   | 登入                                                                       | 菜單                                                                   | 儀表板                                                                           |
| ---------------------------------------------------------------------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| ![root](https://github.com/nepikn/blog/blob/main/screenshots/root.jpg) | ![signin](https://github.com/nepikn/blog/blob/main/screenshots/signin.jpg) | ![menu](https://github.com/nepikn/blog/blob/main/screenshots/menu.jpg) | ![dashboard](https://github.com/nepikn/blog/blob/main/screenshots/dashboard.jpg) |

## API

| 路徑              | 方法   | 用途                 |
| ----------------- | ------ | -------------------- |
| `/auth`           | `POST` | 核對密碼、簽署 token |
| `/auth`           | `GET`  | 驗證、解碼 token     |
| `/post`           | `GET`  | 查詢所有貼文         |
| `/post/reactions` | `GET`  | 查詢所有貼文的回應   |
| `/post/:category` | `GET`  | 查詢特定類別的貼文   |

## 主要技術

- 前端
  - `react` v18
  - `react-router-dom` v6
  - `@mui/material` v6
- 後端
  - `express` v4
- 測試
  - `vitest` v2
  - `@testing-library/react` v16

## 指令

```bash
# 安裝
pnpm install
# 開發
pnpm dev
# 測試
pnpm test
```

## 學習內容

- 調整 Apache 使其
  - 回應對於三級網域的請求
  - 反向代理 `/api` 到後端伺服器的 port
  - 對於無效的請求路徑一律回應 `index.html` 以便前端管理路由

```apache
# /etc/apache2/sites-available/subdomain-ssl.conf
<VirtualHost *:443>
    ServerName blog.unconscious.cc
    VirtualDocumentRoot /var/www/subdomain/%1

    ProxyRequests Off
    ProxyPass /api http://localhost:3001
    ProxyPassReverse /api http://localhost:3001

    <Directory ".">
        RewriteEngine On

        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule . index.html [L]
    </Directory>
    # ...
</VirtualHost>
```

- [JWT 學習歷程](https://hackmd.io/HV43oYz0S5q1z4mDRGK23g?view#JWT)
  - 將用戶 ID 編碼為 JWT
  - 驗證 JWT 完整性

```javascript
// packages/server/src/routers/auth.js
export const auth = express
  .Router()
  .post("", (req, res, next) => {
    const { name, password } = req.body;
    // ...
    const token = jwt.sign({ name }, process.env.JWT_SECRET);

    res.json(token);
  })
  .get("", (req, res, next) => {
    try {
      // ...
      const secret = process.env.JWT_SECRET;
      const decoded = jwt.verify(token, secret);

      res.json(decoded);
    } catch (err) {
      // ...
    }
  });
```

- [React Router 學習歷程](https://hackmd.io/Kic_y5eZQZeM_9MjPJublw?view#React-Router-v626)
  - 以 `RouteObject.lazy` 拆分 bundle
  - 以 `fetcher.Form` 處理無需跳轉的表單

```javascript
// packages/client/src/routes/index.jsx
export default [
  // ...
  {
    path: ":category",
    lazy: async () => {
      const { Category } = await import("./dashboard");
      return {
        element: <Category />,
      };
    },
    // ...
  },
];
```

```jsx
// packages/client/src/components/post.jsx
function Reactions({ reactions, title }) {
  const user = useContext(Auth);
  const fetcher = useFetcher();

  return [
    // ...
  ].map(({ icons, isIconButton, value, ...props }) => {
    // ...
    return (
      <fetcher.Form method="put">
        <input hidden name="title" defaultValue={title} />
        <input hidden name="username" defaultValue={user.name} />
        <input hidden name="reaction" defaultValue={value} />
        {/* ... */}
      </fetcher.Form>
    );
  });
}
```

- [Testing 學習歷程](https://hackmd.io/83yNiSP-RKylyS7dwsNnVw?view)
  - 以 `vitest` 測試

```jsx
// packages/client/test/routes/dashboard/children/category.test.jsx
describe("Post", () => {
  it("changes icons and numbers if reacted", async () => {
    vi.mock("localforage");

    await setup(
      // ...
      async (user) =>
        user.click(
          await screen.findByRole("button", { name: /117/i }),
        ),
    );

    expect(
      await screen.findByRole("button", { name: /118/i }),
    ).toContainElement(screen.getByTestId("EmojiEmotionsIcon"));
  });
});
```

## 展望

- 以 `msw` 模擬後端回應

## 相關資料

- [Heidi-Liu〈用 SPA 架構實作一個部落格〉](https://github.com/heidiliu2020/this-is-codediary/tree/master?tab=readme-ov-file#week221109--1115%E5%89%8D%E7%AB%AF%E6%A1%86%E6%9E%B6%E4%BA%8C)

## 素材

- [RUTVIK PATEL “Mindly - Bloging Website”](https://www.figma.com/community/file/1412677348155758051/mindly-bloging-website)
