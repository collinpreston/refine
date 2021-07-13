---
id: i18n-provider
title: i18n Provider
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

import changeLanguage from '@site/static/img/i18n/changing-language.gif';

refine could support any i18n framework. You just need to create an i18nProvider according to the library you will use.

If you want to add i18n support in the app, refine expects the `i18nProvider` type as follows.

```ts
const i18nProvider = {
    translate: (key: string, params: object) => string,
    changeLocale: (lang: string) => Promise,
    getLocale: () => string,
};
```

`i18nProvider` allows us to set translation features to hooks (`useTranslate`, `useSetLocale`, `useGetLocale`).

-   `useTranslate` shows translation between different languages.
-   `useSetLocale` changes locale at runtime.
-   `useGetLocale` getting current locale.

After creating a `i18nProvider`, you can pass it to the `<Refine>` component.

```tsx title="src/App.tsx"
import { Refine, Resource } from "@pankod/refine";
import dataProvider from "@pankod/refine-simple-rest";

import i18nProvider from "./i18nProvider";

const App: React.FC = () => {
    return (
        <Refine
            dataProvider={dataProvider("https://api.fake-rest.refine.dev")}
            i18nProvider={i18nProvider}
        >
            <Resource name="posts" />
        </Refine>
    );
};
```

:::note
The default refine language is currently English. If you want to use other languages, follow the instructions above. If your application is in English, you don't need to create an i18nProvider.
:::

## Example

Let's add multi-language support using the `react-i18next` framework. At the end of our example, our application supports Turkish and English.

[Refer to react-i18n docs for detailed information &#8594](https://react.i18next.com/getting-started)

### Installation

Run the following command within your project directory to install both `react-i18next` and `i18next` package:

```
npm install react-i18next i18next
```

### Creating i18n Instance

First, we will create an i18n instance using `react-i18next`.

```ts title="src/i18n.ts"
import i18n from "i18next";
import { initReactI18next } from "react-i18next";
import Backend from "i18next-xhr-backend"; // adding lazy loading for translations, more information here: https://react.i18next.com/legacy-v9/step-by-step-guide#2-lazy-loading-translations
import detector from "i18next-browser-languagedetector"; // auto detect the user language, more information here: https://react.i18next.com/legacy-v9/step-by-step-guide#c-auto-detect-the-user-language

i18n.use(Backend)
    .use(detector)
    .use(initReactI18next)
    .init({
        supportedLngs: ["en", "tr"],
        backend: {
            loadPath: "/locales/{{lng}}/{{ns}}.json", // locale files path
        },
        defaultNS: "common",
        fallbackLng: ["en", "tr"],
    });

export default i18n;
```

### Wraping app with React.Suspense

We will import the i18n instance we created and wrap the application with `React.Suspense`.

```tsx title="src/index.tsx"
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

//highlight-next-line
import "./i18n";

ReactDOM.render(
    <React.StrictMode>
        //highlight-start
        <React.Suspense fallback="loading">
            <App />
        </React.Suspense>
        //highlight-end
    </React.StrictMode>,
    document.getElementById("root"),
);
```

:::tip
We use `React.Suspense` because it improves performance by preventing the app from rendering unnecessarily.
:::

### Creating i18n Provider

Next, we will include the i18n instance and create the `i18nProvider` using `react-i18next`.

```tsx title="src/App.tsx"
import { Refine, Resource } from "@pankod/refine";
import dataProvider from "@pankod/refine-simple-rest";
//highlight-next-line
import { useTranslation } from "react-i18next";

import { PostList } from "pages/posts";

const App: React.FC = () => {
    //highlight-next-line
    const { t, i18n } = useTranslation();

    //highlight-start
    const i18nProvider = {
        translate: (key: string, params: object) => t(key, params),
        changeLocale: (lang: string) => i18n.changeLanguage(lang),
        getLocale: () => i18n.language,
    };
    //highlight-end

    return (
        <Refine
            dataProvider={dataProvider("https://api.fake-rest.refine.dev")}
            //highlight-next-line
            i18nProvider={i18nProvider}
        >
            <Resource name="posts" list={PostList} />
        </Refine>
    );
};
```

After we pass the i18nProvider to the `<Refine>` component, translation hooks(`useTranslate`, `useSetLocale`, `useGetLocale`) are fully ready to use.

### Adding Translations Files

Before we get started, let's look at the translations that refine uses in components.

```json
{
    "pages": {
        "login": {
            "login": "Login",
            "signup": "Sign up",
            "username": "Username",
            "password": "Password",
            "remember": "Remember me",
            "forgotPassword": "Forgot password?",
            "noAccount": "Still no account? Please go to"
        },
        "error": {
            "info": "You may have forgotten to add the {{action}} component to {{resource}} resource.",
            "404": "Sorry, the page you visited does not exist.",
            "resource404": "Are you sure you have created the {{resource}} resource.",
            "backHome": "Back Home"
        }
    },
    "buttons": {
        "create": "Create",
        "save": "Save",
        "logout": "Logout",
        "delete": "Delete",
        "edit": "Edit",
        "cancel": "Cancel",
        "confirm": "Are you sure?",
        "filter": "Filter",
        "clear": "Clear",
        "refresh": "Refresh",
        "show": "Show",
        "undo": "Undo",
        "import": "Import",
        "clone": "Clone"
    },
    "warnWhenUnsavedChanges": "Are you sure you want to leave? You have unsaved changes.",
    "notifications": {
        "success": "Successful",
        "error": "Error (status code: {{statusCode}})",
        "undoable": "You have {{seconds}} seconds to undo",
        "createSuccess": "Successfully created {{resource}}",
        "createError": "There was an error creating {{resource}} (status code: {{statusCode}})",
        "deleteSuccess": "Successfully deleted {{resource}}",
        "deleteError": "Error when deleting {{resource}} (status code: {{statusCode}})",
        "editSuccess": "Successfully edited {{resource}}",
        "editError": "Error when editing {{resource}} (status code: {{statusCode}})"
    },
    "loading": "Loading",
    "tags": {
        "clone": "Clone"
    },
    "dashboard": {
        "title": "Dashboard"
    }
}
```

All components of refine supports i18n. If you want to change the refine component texts, you can create your own translation file with reference to the keys above.

Next, let's add the language files:

```
|-- public
|   |-- locales
|       |-- en
|       |   |-- common.json
|       |-- tr
|           |-- common.json
|-- src
|-- package.json
|-- tsconfig.json
```

<Tabs
defaultValue="en"
values={[{ label: "English", value: "en" }, { label: "Turkish", value: "tr" }]}>
<TabItem value="en">

```json title="/locales/en/common.json"
{
    "pages": {
        "login": {
            "login": "Login",
            "signup": "Sign up",
            "username": "Username",
            "password": "Password",
            "remember": "Remember me",
            "forgotPassword": "Forgot password?",
            "noAccount": "Still no account? Please go to"
        },
        "error": {
            "info": "You may have forgotten to add the {{action}} component to {{resource}} resource.",
            "404": "Sorry, the page you visited does not exist.",
            "resource404": "Are you sure you have created the {{resource}} resource.",
            "backHome": "Back Home"
        }
    },
    "buttons": {
        "create": "Create",
        "save": "Save",
        "logout": "Logout",
        "delete": "Delete",
        "edit": "Edit",
        "cancel": "Cancel",
        "confirm": "Are you sure?",
        "filter": "Filter",
        "clear": "Clear",
        "refresh": "Refresh",
        "show": "Show",
        "undo": "Undo",
        "import": "Import",
        "clone": "Clone"
    },
    "warnWhenUnsavedChanges": "Are you sure you want to leave? You have unsaved changes.",
    "notifications": {
        "success": "Successful",
        "error": "Error (status code: {{statusCode}})",
        "undoable": "You have {{seconds}} seconds to undo",
        "createSuccess": "Successfully created {{resource}}",
        "createError": "There was an error creating {{resource}} (status code: {{statusCode}})",
        "deleteSuccess": "Successfully deleted {{resource}}",
        "deleteError": "Error when deleting {{resource}} (status code: {{statusCode}})",
        "editSuccess": "Successfully edited {{resource}}",
        "editError": "Error when editing {{resource}} (status code: {{statusCode}})"
    },
    "loading": "Loading",
    "tags": {
        "clone": "Clone"
    },
    "dashboard": {
        "title": "Dashboard"
    },
    "posts": {
        "posts": "Posts",
        "fields": {
            "title": "Title",
            "category": "Category"
        },
        "titles": {
            "list": "Posts"
        }
    },
    "table": {
        "actions": "Actions"
    }
}
```

</TabItem>
<TabItem value="tr">

```json title="/locales/tr/common.json"
{
    "pages": {
        "login": {
            "login": "Giriş",
            "signup": "Kayıt",
            "username": "Kullanıcı Adı",
            "password": "Şifre",
            "remember": "Beni Hatırla",
            "forgotPassword": "Şifremi unuttum",
            "noAccount": "Hala üye olmadın mı? Hadi tıkla"
        },
        "error": {
            "info": "{{action}} sayfasını {{resource}} kaynağına eklemeyi unutmuş olabilirsiniz.",
            "404": "Üzgünüz, ziyaret ettiğiniz sayfa mevcut değil.",
            "resource404": "{{resource}} kaynağını oluşturduğunuzdan emin misiniz?",
            "backHome": "Ana sayfaya geri dön"
        }
    },
    "buttons": {
        "create": "Oluştur",
        "save": "Kaydet",
        "logout": "Çıkış yap",
        "delete": "Sil",
        "edit": "Düzenle",
        "cancel": "İptal",
        "confirm": "Emin misiniz?",
        "filter": "Filtrele",
        "clear": "Temizle",
        "refresh": "Yenile",
        "show": "Göster",
        "undo": "Geri Al",
        "import": "Yükle",
        "clone": "Kopya"
    },
    "warnWhenUnsavedChanges": "Kaydedilmemiş değişiklikleriniz var. Ayrılmak istediğinizden emin misiniz?",
    "notifications": {
        "success": "Başarılı",
        "error": "Hata (hata kodu: {{statusCode}})",
        "undoable": "Geri almak için {{seconds}} saniyeniz var",
        "createSuccess": "{{resource}} başarıyla oluşturuldu",
        "createError": "{{resource}} oluşturulurken hata (hata kodu: {{statusCode}})",
        "deleteSuccess": "{{resource}} başarıyla silindi",
        "deleteError": "{{resource}} silinirken hata (hata kodu: {{statusCode}})",
        "editSuccess": "{{resource}} başarıyla düzenlendi",
        "editError": "{{resource}} düzenlenirken hata (hata kodu: {{statusCode}})"
    },
    "loading": "Yükleniyor",
    "tags": {
        "clone": "Kopya"
    },
    "dashboard": {
        "title": "Panel"
    },
    "posts": {
        "posts": "Gönderiler",
        "fields": {
            "title": "Başlık",
            "category": "Kategori"
        },
        "titles": {
            "list": "Gönderiler"
        }
    },
    "table": {
        "actions": "İşlemler"
    }
}
```

</TabItem>
</Tabs>

:::tip
We can override refine's default texts by changing from the above common.json files.
:::

### Changing The Locale

Next, we will create a `<Header>` component. This component will allow us to change the language.

```tsx title="src/components/header.tsx"
import {
    AntdLayout,
    Button,
    Menu,
    Icons,
    Dropdown,
    useGetLocale,
    useSetLocale,
} from "@pankod/refine";
import { useTranslation } from "react-i18next";
import { ILanguage } from "interfaces";

const { TranslationOutlined } = Icons;

const languages: Record<string, ILanguage> = {
    en: {
        title: "English",
        flag: "🏴󠁧󠁢󠁥󠁮󠁧󠁿",
    },
    tr: {
        title: "Türkçe",
        flag: "🇹🇷",
    },
};

export const Header: React.FC = () => {
    const { i18n } = useTranslation();
    const locale = useGetLocale();
    const changeLanguage = useSetLocale();

    const menu = (
        <Menu selectedKeys={[locale()]}>
            {i18n.languages?.sort().map((lang) => (
                <Menu.Item
                    key={lang}
                    onClick={() => changeLanguage(lang)}
                    icon={
                        <span style={{ marginRight: 8 }}>
                            {languages[lang].flag}
                        </span>
                    }
                >
                    {languages[lang].title}
                </Menu.Item>
            ))}
        </Menu>
    );

    return (
        <AntdLayout.Header
            style={{
                padding: "0px 24px 0px 0px",
                height: "48px",
                backgroundColor: "#FFF",
            }}
        >
            <div
                style={{
                    display: "flex",
                    justifyContent: "flex-end",
                    height: "100%",
                    alignItems: "center",
                }}
            >
                <Dropdown overlay={menu}>
                    <Button
                        type="text"
                        size="large"
                        style={{ height: "100%" }}
                        icon={<TranslationOutlined />}
                        onClick={(e) => e.preventDefault()}
                    />
                </Dropdown>
            </div>
        </AntdLayout.Header>
    );
};
```

```ts title="interfaces/index.d.ts"
export interface ILanguage {
    title: string;
    flag: string;
}
```

<br/>

Now, we will pass `<Header>` to the `<Refine>` component as a prop.

```tsx title="src/App.tsx"
import { Refine, Resource } from "@pankod/refine";
import dataProvider from "@pankod/refine-simple-rest";
import { useTranslation } from "react-i18next";
import "./i18n";

import { PostList } from "pages/posts";

//highlight-next-line
import { Header } from "components";

const App: React.FC = () => {
    const { t, i18n } = useTranslation();

    const i18nProvider = {
        translate: (key: string, params: object) => t(key, params),
        changeLocale: (lang: string) => i18n.changeLanguage(lang),
        getLocale: () => i18n.language,
    };

    return (
        <Refine
            dataProvider={dataProvider("https://api.fake-rest.refine.dev")}
            i18nProvider={i18nProvider}
            //highlight-next-line
            Header={Header}
        >
            <Resource name="posts" list={PostList} />
        </Refine>
    );
};
```

<br />

Finally, we will create `<PostList>` page and then we will translate texts using `useTranslate`.

```tsx title="src/App.tsx"
import {
    List,
    Table,
    TextField,
    useTable,
    Space,
    EditButton,
    ShowButton,
    useMany,
    //highlight-next-line
    useTranslate,
} from "@pankod/refine";

import { IPost, ICategory } from "interfaces";

export const PostList: React.FC = () => {
    //highlight-next-line
    const translate = useTranslate();
    const { tableProps } = useTable<IPost>();

    const categoryIds =
        tableProps?.dataSource?.map((item) => item.category.id) ?? [];
    const { data, isLoading } = useMany<ICategory>("categories", categoryIds, {
        enabled: categoryIds.length > 0,
    });

    return (
        //highlight-next-line
        <List>
            <Table {...tableProps} key="id">
                <Table.Column key="id" dataIndex="id" title="ID" />
                <Table.Column
                    key="title"
                    dataIndex="title"
                    //highlight-next-line
                    title={translate("posts.fields.title")}
                />
                <Table.Column
                    dataIndex={["category", "id"]}
                    key="category.id"
                    //highlight-next-line
                    title={translate("posts.fields.category")}
                    render={(value) => {
                        if (isLoading) {
                            return <TextField value="Loading..." />;
                        }

                        return (
                            <TextField
                                value={
                                    data?.data.find((item) => item.id === value)
                                        ?.title
                                }
                            />
                        );
                    }}
                />
                <Table.Column<IPost>
                    //highlight-next-line
                    title={translate("table.actions")}
                    dataIndex="actions"
                    key="actions"
                    render={(_value, record) => (
                        <Space>
                            <EditButton size="small" recordItemId={record.id} />
                            <ShowButton size="small" recordItemId={record.id} />
                        </Space>
                    )}
                />
            </Table>
        </List>
    );
};
```

```ts title="interfaces/index.d.ts"
export interface ICategory {
    id: string;
    title: string;
}

export interface IPost {
    id: string;
    title: string;
    content: string;
    status: "published" | "draft" | "rejected";
    category: ICategory;
}
```

<br/>

<div style={{textAlign: "center"}}>
    <img src={changeLanguage} />
</div>

## Live Codesandbox Example

<iframe src="https://codesandbox.io/embed/refine-i18n-example-xvk6w?autoresize=1&fontsize=14&module=%2Fsrc%2FApp.tsx&theme=dark&view=preview"
    style={{width: "100%", height:"80vh", border: "0px", borderRadius: "8px", overflow:"hidden"}}
     title="refine-i18n-example"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>