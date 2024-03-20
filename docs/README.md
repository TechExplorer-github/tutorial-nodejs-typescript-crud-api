# Node.js and TypeScript Tutorial: Build a CRUD API

## 環境構築

Node.js のプロジェクトの初期化

```sh
npm init -y
```

API の開発で利用するライブラリをインストール

```sh
npm i express dotenv cors helmet
```

TypeScript のインストール
-D は--save-dev の略

```sh
npm i -D typescript
```

ライブラリの型定義をインストール

```sh
npm i -D @types/node @types/express @types/dotenv @types/cors @types/helmet
```

TypeScript プロジェクトの初期設定ファイル(tsconfig.json)を作成

```sh
npx tsc --init
```

環境ファイルを作成

```sh
# Linuxの場合
touch .env

# Windowsの場合
New-Item test.txt
```

.env に PORT を追記

```sh
PORT=7000
```

ソースは src の下に作成していくのでフォルダを作成

```sh
mkdir src
```

API のエントリーポイントの index.ts を作成

```sh
# Linuxの場合
touch ./src/index.ts

# Windowsの場合
New-Item ./src/index.ts
```

## エントリーポイントの実装

src/index.ts を下記のように記述

```ts
import * as dotenv from "dotenv";
import express from "express";
import cors from "cors";
import helmet from "helmet";

/**
 * App Variables
 */
dotenv.config();

if (!process.env.PORT) {
  console.log(`Not Found PORT`);
  process.exit(1);
}

const PORT: number = parseInt(process.env.PORT as string, 10);

const app = express();

/**
 *  App Configuration
 */
app.use(helmet());
app.use(cors());
app.use(express.json());

/**
 * Server Activation
 */
app.listen(PORT, () => {
  console.log(`Listening on port ${PORT}`);
});
```

ts-node-dev のライブラリをインストール  
ts ファイルを js ファイルにコンパイルすることなく、起動することができ、さらに監視モードで素早く再起動が使用できる

```sh
npm i -D ts-node-dev
```

package.json の scripts に下記を追加

```json
  "scripts": {
    "dev": "ts-node-dev --respawn --pretty --transpile-only src/index.ts"
  },
```

下記のコマンドで実行

```sh
npm run dev
```

## TypeScript の Interfaces を使用してモデルデータを定義

items のフォルダを作成

```sh
mkdir src/items
```

item の interface のファイルを作成

```sh
# Linuxの場合
touch src/items/item.interface.ts

# Windowsの場合
New-Item src/items/item.interface.ts
```

src/items/item.interface.ts は下記のように記述

```ts
export interface BaseItem {
  name: string;
  price: number;
  description: string;
  image: string;
}

export interface Item extends BaseItem {
  id: number;
}
```

続いて items の interface のファイルを作成

```sh
# Linuxの場合
touch src/items/items.interface.ts

# Windowsの場合
New-Item src/items/items.interface.ts
```

src/items/items.interface.ts は下記のように記述

```ts
import { Item } from "./item.interface";

export interface Items {
  [key: number]: Item;
}
```

## サービスを実装

items の service のファイルを作成

```sh
# Linuxの場合
touch src/items/items.service.ts

# Windowsの場合
New-Item src/items/items.service.ts
```

src/items/items.service.ts は下記のように記述  
データはデータベースを使用しないのでインメモリに格納

```ts
import { BaseItem, Item } from "./item.interface";
import { Items } from "./items.interface";

/**
 * In-Memory Store
 */
let items: Items = {
  1: {
    id: 1,
    name: "Burger",
    price: 599,
    description: "Tasty",
    image: "https://cdn.auth0.com/blog/whatabyte/burger-sm.png",
  },
  2: {
    id: 2,
    name: "Pizza",
    price: 299,
    description: "Cheesy",
    image: "https://cdn.auth0.com/blog/whatabyte/pizza-sm.png",
  },
  3: {
    id: 3,
    name: "Tea",
    price: 199,
    description: "Informative",
    image: "https://cdn.auth0.com/blog/whatabyte/tea-sm.png",
  },
};

/**
 * Service Methods
 */
export const findAll = async (): Promise<Item[]> => Object.values(items);

export const find = async (id: number): Promise<Item> => items[id];

export const create = async (newItem: BaseItem): Promise<Item> => {
  const id = new Date().valueOf();

  items[id] = {
    id,
    ...newItem,
  };

  return items[id];
};

export const update = async (
  id: number,
  itemUpdate: BaseItem
): Promise<Item | null> => {
  const item = await find(id);

  if (!item) {
    return null;
  }

  items[id] = { id, ...itemUpdate };

  return items[id];
};

export const remove = async (id: number): Promise<null | void> => {
  const item = await find(id);

  if (!item) {
    return null;
  }

  delete items[id];
};
```

## コントローラを実装

items の router のファイルを作成

```sh
# Linuxの場合
touch src/items/items.router.ts

# Windowsの場合
New-Item src/items/items.router.ts
```

src/items/items.router.ts は下記のように記述

```ts
import express, { Request, Response } from "express";
import * as ItemService from "./items.service";
import { BaseItem, Item } from "./item.interface";

/**
 * Router Definition
 */
export const itemsRouter = express.Router();

/**
 * Controller Definitions
 */
// GET items
itemsRouter.get("/", async (req: Request, res: Response) => {
  try {
    const items: Item[] = await ItemService.findAll();

    res.status(200).send(items);
  } catch (e: any) {
    res.status(500).send(e.message);
  }
});

// GET items/:id
itemsRouter.get("/:id", async (req: Request, res: Response) => {
  const id: number = parseInt(req.params.id, 10);

  try {
    const item: Item = await ItemService.find(id);

    if (item) {
      return res.status(200).send(item);
    }

    res.status(404).send("item not found");
  } catch (e: any) {
    res.status(500).send(e.message);
  }
});

// POST items
itemsRouter.post("/", async (req: Request, res: Response) => {
  try {
    const item: BaseItem = req.body;

    const newItem = await ItemService.create(item);

    res.status(201).json(newItem);
  } catch (e: any) {
    res.status(500).send(e.message);
  }
});

// PUT items/:id
itemsRouter.put("/:id", async (req: Request, res: Response) => {
  const id: number = parseInt(req.params.id, 10);

  try {
    const itemUpdate: Item = req.body;

    const existingItem: Item = await ItemService.find(id);

    if (existingItem) {
      const updatedItem = await ItemService.update(id, itemUpdate);
      return res.status(200).json(updatedItem);
    }

    const newItem = await ItemService.create(itemUpdate);

    res.status(201).json(newItem);
  } catch (e: any) {
    res.status(500).send(e.message);
  }
});

// DELETE items/:id
itemsRouter.delete("/:id", async (req: Request, res: Response) => {
  try {
    const id: number = parseInt(req.params.id, 10);
    await ItemService.remove(id);

    res.sendStatus(204);
  } catch (e: any) {
    res.status(500).send(e.message);
  }
});
```

src/index.ts に下記の router を追加

```ts
import { itemsRouter } from "./items/items.router";

app.use("/api/menu/items", itemsRouter);
```

## 動作確認

下記のコマンドで起動

```sh
npm run dev
```

API はコマンドプロンプトで curl を利用して確認

Get all items: `curl http://localhost:7000/api/menu/items -i`

Get an item: `curl http://localhost:7000/api/menu/items/2 -i`

Add an item:

```sh
curl -X POST -H 'Content-Type: application/json' -d '{
"name": "Salad",
"price": 499,
"description": "Fresh",
"image": "https://images.ctfassets.net/23aumh6u8s0i/5pnNAeu0kev0P5Neh9W0jj/5b62440be149d0c1a9cb84a255662205/whatabyte_salad-sm.png"
}' http://localhost:7000/api/menu/items -i

curl http://localhost:7000/api/menu/items/ -i
```

Update an item:

```sh
curl -X PUT -H 'Content-Type: application/json' -d '{
"name": "Spicy Pizza",
"price": 599,
"description": "Blazing Good",
"image": "https://images.ctfassets.net/23aumh6u8s0i/2x1D2KeepKoZlsUq0SEsOu/bee61947ed648848e99c71ce22563849/whatabyte_pizza-sm.png"
}' http://localhost:7000/api/menu/items/2 -i

curl http://localhost:7000/api/menu/items/2 -i
```

Delete an item:

```sh
curl -X DELETE http://localhost:7000/api/menu/items/2 -i

curl http://localhost:7000/api/menu/items/ -i
```
