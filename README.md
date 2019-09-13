# NodeJS Express

```
npm init
npm install --save-exact express@4.16.4 @types/express@4.16.0 @okta/jwt-verifier@0.0.14 express-bearer-token@2.2.0 tsc@1.20150623.0 typescript@3.1.3 typeorm@0.2.8 sqlite3@4.0.3 cors@2.8.4 @types/cors@2.8.4
```
```
vim tsconfig.json
```
```
{
  "compilerOptions": {
    "target": "es6",
    "module": "commonjs",
    "outDir": "dist",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  },
  "include": [
    "src/**/*.ts"
  ],
  "exclude": [
    "node_modules"
  ]
}
```
```
vim model.ts
```
```
import {Entity, PrimaryGeneratedColumn, Column, createConnection, Connection, Repository} from 'typeorm';

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  sku: string;

  @Column('text')
  description: string;

  @Column()
  price: number;

  @Column()
  stock: number;
}

let connection:Connection;

export async function getProductRepository(): Promise<Repository<Product>> {
  if (connection===undefined) {
    connection = await createConnection({
      type: 'sqlite',
      database: 'myangularapp',
      synchronize: true,
      entities: [
        Product
      ],
    });
  }
  return connection.getRepository(Product);
}
```
```
vim product.ts
```
```
import { NextFunction, Request, Response, Router } from 'express';
import { getProductRepository, Product } from './model';

export const router: Router = Router();

router.get('/product', async function (req: Request, res: Response, next: NextFunction) {
  try {
    const repository = await getProductRepository();
    const allProducts = await repository.find();
    res.send(allProducts);
  }
  catch (err) {
    return next(err);
  }
});

router.get('/product/:id', async function (req: Request, res: Response, next: NextFunction) {
  try {
    const repository = await getProductRepository();
    const product = await repository.find({id: req.params.id});
    res.send(product);
  }
  catch (err) {
    return next(err);
  }
});

router.post('/product', async function (req: Request, res: Response, next: NextFunction) {
  try {
    const repository = await getProductRepository();
    const product = new Product();
    product.name = req.body.name;
    product.sku = req.body.sku;
    product.description = req.body.description;
    product.price = Number.parseFloat(req.body.price);
    product.stock = Number.parseInt(req.body.stock);

    const result = await repository.save(product);
    res.send(result);
  }
  catch (err) {
    return next(err);
  }
});

router.post('/product/:id', async function (req: Request, res: Response, next: NextFunction) {
  try {
    const repository = await getProductRepository();
    const product = await repository.findOne({id: req.params.id});
    product.name = req.body.name;
    product.sku = req.body.sku;
    product.description = req.body.description;
    product.price = Number.parseFloat(req.body.price);
    product.stock = Number.parseInt(req.body.stock);

    const result = await repository.save(product);
    res.send(result);
  }
  catch (err) {
    return next(err);
  }
});

router.delete('/product/:id', async function (req: Request, res: Response, next: NextFunction) {
  try {
    const repository = await getProductRepository();
    await repository.delete({id: req.params.id});
    res.send('OK');
  }
  catch (err) {
    return next(err);
  }
});
```

```
npx tsc
```
```
node dist/server.js
```
