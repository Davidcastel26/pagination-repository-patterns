# pagination-repository-patterns
This a quick setup in order to create the pagination

## Dir structure 

- 1 🏃🏽‍♂️ into domain we follow this structured 
```
└── 📁domain
    └── 📁datasources
      └── 📁product-datasource
        └── produc.datasource.ts
    └── 📁dto
      └── 📁products-dto
          └── 📁product
            └── find-product.dto.ts
      └── 📁shared-dto
          └── pagination.dto.ts
    └── 📁entities
        └── 📁products-entities
            └── product.entities.ts
    └── 📁repositories
        └── 📁products-repository
            └── index.ts
            └── product-repository.ts
    └── 📁use-cases
        └── 📁products-cases
            └── index.ts
            └── 📁product-cases
                └── get-all-products.ts
```
- 2 😵‍💫 this is infrastructure
```
└── 📁infrastructure
    └── 📁datasource
        └── 📁product-datasource-inf
            └── index.ts
            └── product-datasource.impl.ts
    └── 📁repositories
        └── 📁products-repositories-inf
            └── index.ts
            └── product.repository.impl.ts   
```
- 3 🤓 and this is in to Presentation
```
└── 📁presentation
    └── 📁orders
        └── 📁products
            └── product.controller.ts
            └── product.routes.ts
    └── 📁routes
        └── index.ts
        └── microsoft.ts
    └── routes.ts
    └── server.ts
```

- 😵‍💫 Feeling lost? dont worry now take a look the code how to start with this:

into /product.datasource.ts
--
- we set the paginationDto as parameter

```ts
export abstract class ProductDatasource {
    abstract getAll(paginationDto: PaginationDto): Promise<PaginationDto>
}
```
into /dto
--
- we set the pagintaiton dto
```ts
export class PaginationDto {
    
    private constructor(
        public readonly page : number,
        public readonly limit : number,   
        public  total: number| any,
        public  next: string | null,
        public  prev: string | null,
        public readonly data: any[] 
    ){}

    static paginationPage(
        page: number = 1,
        limit: number = 10,
        total: number | any,
        data: any[],
        baseUrl: string | string[]
      ):  [string?, PaginationDto?] {
    

    if (isNaN(page) || isNaN(limit) || page <= 0 || limit <= 0) {
        throw new Error('Invalid pagination parameters');
      }

        console.log(total + ' es este total')
        let next = `${baseUrl}?page=${page + 1}&limit=${limit}`;
        let prev = page - 1 > 0 ? `${baseUrl}?page=${page - 1}&limit=${limit}` : null;


        return  [undefined, new PaginationDto(page, limit, total, next, prev, data)]

    }
}
```
this is into /repositories
--
- its like the datasource in domain
```ts
  export abstract class ProductRepository {
    abstract getAll(paginationDto: PaginationDto): Promise<PaginationDto>
}
```

- now setting the use case 
```ts
export class GetAllProductCase implements GetAllProductUseCase {
  constructor(private readonly repository: ProductRepository) {}

  execute(dto: PaginationDto): Promise<ProductEntity[]> {
    return this.repository.getAll(dto).then((paginationDto) => paginationDto.data);
  }
}
```
now we follow up into infrastructure
--
- we set the repository for product dto
```ts
export class ProductRepositoryImpl implements ProductRepository{

    constructor(
        private readonly datasource: ProductDatasource
    ){}

    getAll(paginationDto: PaginationDto): Promise<PaginationDto> {
        return this.datasource.getAll(paginationDto)
    }

}
```

- time to build the datasource product implementation

```ts
export class ProductsDataSourceImpl implements ProductDatasource {
    async getAll(paginationDto: PaginationDto): Promise<PaginationDto> {
      const { limit, page } = paginationDto;
      const offset = (page - 1) * limit;
      let [totalResult, getAllData] = await Promise.all([
        prismadb.products.count({
          select: {
            _all: true,
          },
        }),
        prismadb.products.findMany({
          include: {
            _count: true,
            category: true,
            priceListOnProducts: true,
          },
          take: limit,
          skip: offset,
        }),
      ]);
    
      const products = getAllData.map((data) => ProductEntity.fromObject(data));
      const baseUrl =  `${envs.WEBSERVICE_URL}/${envs.SERVER_NAME}/${envs.API_VERSION}/products`;
    
      console.log(totalResult._all); // This will log the total count of products
    
      const [error, paginationDto_] = PaginationDto.paginationPage(page, limit, totalResult._all, products, baseUrl);
    
      if (error) {
        throw new Error(error);
      }
    
      if (!paginationDto_) {
        throw new Error('Failed to create pagination');
      }
    
      return paginationDto_;
    }

}
```
and finally we could go to presentation
--
now into the product controller
```ts
export class ProductsController {

    constructor (
        private readonly productRepository: ProductRepository
    ){}

    private handleError = ( error: unknown, res: Response) => {

        if( error instanceof CustomError){
            return res.status(error.statusCode).json({error: error.message})
        }

        return res.status(500).json({ error: 'Internal Server Error'})
    }
  public getAll = async(
        req: Request,
        res: Response,
        next: NextFunction
    ) => {

        const { page = 1, limit = 10 } = req.query; // send the page and limit from query


        
        const [error, paginationDto_] = PaginationDto.paginationPage( 
            +page, // default 1 
            +limit, // default 10
            0, // Replace with the actual total count
            [], // Replace with the actual data array this returns data with the fechecin in the /product-datasource.impl.ts
            `${envs.WEBSERVICE_URL}/${envs.SERVER_NAME}/${envs.API_VERSION}/products/all`, // this should be '' an empty string and be updated with the /product-datasource.impl.ts
          );

        
        if (error) { // review if there is any error if not next 
            return this.handleError(error, res);
        }

        if (!paginationDto_) { // review if there is data into paginationDto_ 
            return this.handleError('Failed to create pagination', res); // return a string with the rest out put
        }

        // new GetAllProductCase(this.productRepository): This line creates a new instance of the GetAllProductCase class, which is responsible for the business logic of fetching all products. The productRepository instance is passed as a dependency to the constructor.
        new GetAllProductCase(this.productRepository)
            .execute(paginationDto_) //.execute(paginationDto_): This calls the execute method of the GetAllProductCase instance, passing the paginationDto_ object as an argument. The execute method is likely responsible for calling the getAll method of the repository and performing any necessary operations on the fetched data.
            .then((data) => {  // This is a Promise callback that will be executed when the execute method successfully resolves with the fetched data. The fetched data is passed as the data parameter to this callback function.
                const response = { 
                page: paginationDto_?.page, //page: paginationDto_?.page: This sets the page property of the response object with the page value from the paginationDto_ object. The optional chaining operator (?.) is used to handle the case where paginationDto_ might be undefined or null.
                limit: paginationDto_?.limit, // limit: paginationDto_?.limit: Similar to the page property, this sets the limit property of the response object with the limit value from the paginationDto_ object.
                total: paginationDto_?.total, // total: paginationDto_?.total: This sets the total property of the response object with the total value from the paginationDto_ object, which represents the total count of products.
                next: paginationDto_?.next, // next: paginationDto_?.next: This sets the next property of the response object with the URL for the next page of results, retrieved from the paginationDto_ object.
                prev: paginationDto_?.prev, // prev: paginationDto_?.prev: This sets the prev property of the response object with the URL for the previous page of results, retrieved from the paginationDto_ object.
                results: data, //results: data: This sets the results property of the response object with the fetched data (data) returned from the execute method.
                };
                res.json(response);
            })
            .catch((error) => this.handleError(error, res));


    }
```
well done!
--
you can use it and try it into products or make the same for any oter route

Teka a look into the [repository](http://10.111.102.10:3000/desa03/lacarreta_backend/src/branch/development)

---
⭐️ From [@Davidcastel26](https://github.com/Davidcastel26) 
