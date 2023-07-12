# Configuraciones para las roles e la app

### Creacion de Controller

Realizamos la creacion de controlador en la ruta `src/apps/modulo/modulo.controller.ts`, el decorador `@ApiTags('modulos')` permite mostrar los endpoints en swagger, el decorador `@controller('modulo')` permite crear el controlador. finalmente se exporta el controlador.

```ts title="src/apps/modulo/modulo.controller.ts"
import { Body, Controller, Delete, Get, HttpException, HttpStatus, Param, Post, Put, UsePipes, ValidationPipe } from "@nestjs/common";
import { ApiTags } from "@nestjs/swagger";
import { CreateRolAppDto} from "./dto/create-rolapp.dto";
import { RolAppService} from "./rolapp.service";
import { AppsService } from "../apps.service";

@ApiTags('Rol-App')
@Controller('Rol-App')
export class RolAppContorller{
    constructor(private readonly serviceRolApp:RolAppService,
    ){}
  //resto del codigo como @Get(), @Post(), @Put(), @Delete(), etc
  }
```

### Metodo Get

El decorador `@Get()` indica que se va ejecutarme un endpoint get, la funcion retorna los datos retuperados de la base de datos asia el cliente. 
![appass-get](/img/rapp-get.png) 

```ts title="src/apps/modulo/modulo.controller.ts"
  @Get()
    async findAll(){
        return await this.serviceRolApp.findAll()
  }
```

### Metodo Get por ID

Esta funcion permitira el filtro de una aplicacion desde su id, returna los datos recuperados de la base de datos hacia el cliente, si embargo no retorna a aquellos aplicaciones que estan con estados elimiando `if(modulo.isDeleted)`, simplemente devuelve el error de mensaje `throw new HttpException('no se encuentra el modulo',HttpStatus.NOT_FOUND)`.
![appas-get](/img/rapp-get-i.png)
```ts title="src/apps/modulo/modulo.controller.ts"
    @Get(':id')
    async findById(@Param('id') id:string){
        const modulo = await this.serviceRolApp.findById(id)
        if(modulo.isDeleted){
            throw new HttpException('no se encuentra el permiso',HttpStatus.NOT_FOUND)
        }
        return modulo
    }
```

### Metodo Post

Esta funcion retorna y registra los datos a la base de datos insertados por el endpoint post, recive como parametro la configuracion en dto. Los descoradores `@UsePipes(new ValidationPipe())` permite mostras los mensajes de errorres que se configuro en dto.
![apr-get](/img/rapp-post.png)
```ts title="src/apps/modulo/modulo.controller.ts"
  @Post('/create')
  @UsePipes(new ValidationPipe())
  async create(@Body() body:CreateRolAppDto){
        return this.serviceRolApp.create(body)
  }
```

### Metodo Put

Esta funcion returna y permite realizar los cambios de campos del modulo, recive en su paramentro la configuracion realizada en dto. Los descoradores `@UsePipes(new ValidationPipe())` permite mostras los mensajes de errorres que se configuro en dto..
![api-get](/img/rapp-put.png)
```ts title="src/apps/modulo/modulo.controller.ts"
  @Put('/update:id')
    @UsePipes(new ValidationPipe())
    async update(@Param('id') id: string, @Body() body:CreateRolAppDto)
    {
        return await this.serviceRolApp.update(id,body)
    }
```

### Metodo Delete

Esta funcion permite eleminar los datos de modulo, la sentencia `throw new HttpException('modulo eliminada',HttpStatus.OK)` retorna un mensaje de exito de eliminacion, si embargo esta funcion no elimana los datos del modulo en la base de datos, solo cambia el estado de eliminacion.
![ggsh-get](/img/rapp-delete.png)
```ts title="src/apps/modulo/modulo.controller.ts"
  @Delete('/delete:id')
    async delete(@Param('id') id:string){
       await this.serviceRolApp.delete(id)
       throw new HttpException('rol eliminado',HttpStatus.OK)

  }
```

### Module

En este aparato se crea los modules, los `imports` importa los modelos, el `MongooseModule.forFeature` sirve para importar modelos al moongose, como es este caso el modelo de modulo, con `name: Modulo.name` hace referncia al nombre de la clase module y `schema: ModuloSchema` importa la esquema del modelo donde se esta trabajando. El `contorllers` importa los controladores que sean necesarios, en este caso `ModuloContorller` esta importando el controlador de modulo. El`ModuloService` importa los servicios que se desea, en este caso `AppsService` esta importando los servicios de modulo.

```ts title="src/apps/modulo/modulo.module.ts"
import { RolAppContorller } from "./rolapp.controller";
import { RolAppService } from "./rolapp.service";
import { MongooseModule, Schema } from "@nestjs/mongoose";
import { RolApp, RolAppSchema } from "./schema/rolapp.schema";
import { PermisosApp, PermisosAppSchema } from "../permisosapp/schema/permisosapp.schema";
import { App, AppSchema } from "../schema/apps.schema";

@Module({
    imports:[MongooseModule.forFeature([
        {
            name:RolApp.name,
            schema:RolAppSchema
        }
        ,        {
            name:App.name,
            schema:AppSchema
        },
        {
            name:PermisosApp.name,
            schema:PermisosAppSchema
        }
    ])
],
    controllers:[RolAppContorller],
    providers:[RolAppService]
})
export class RolAppModule{}
```

### Service

El decorador `@Injectable()` indica la inyeccion de dependencias. En el constructor se esta inyectando el modelo de las modulo obteniendo de la esquema `ModuloDocument`.

```ts title="src/apps/modulo/modulo.service.ts"
import { HttpException, HttpStatus, Injectable } from "@nestjs/common";
import { InjectModel } from "@nestjs/mongoose";
import { RolApp, RolAppDocument } from "./schema/rolapp.schema";
import { Model } from "mongoose";
import { CreateRolAppDto } from "./dto/create-rolapp.dto";
import { App, AppDocument } from "../schema/apps.schema";
import { PermisosApp, PermisosAppDocument } from "../permisosapp/schema/permisosapp.schema";

@Injectable()
export class RolAppService{
    constructor(@InjectModel(RolApp.name) private readonly RolAppModel:Model<RolAppDocument>,
    @InjectModel(App.name) private readonly appModel:Model<AppDocument>,
    @InjectModel(PermisosApp.name) private readonly permisoModel: Model<PermisosAppDocument>
    )
    {}
  // El resto del código las funciones de Get Post Put, etc
}
```

### Funcion findAll()

La funcion retorna todos los datos del modulo de la base de datos, si embargo no retorna los modulos que estan con estado eliminado, la ejecucion realiza a traves del modelo `moduloModel`.

```ts title="src/apps/modulo/modulo.service.ts"
    async findAll(){
        return await this.RolAppModel.find()
    }
```

### Funcion findById(id:string)

Esta funcion retorna la informacion de un modulo por el id, pero siempre en cuando no se presente alguna excepcion, por ejemplo puede ver casos en que la id sea incorrecto, si fuera el caso devuelve un error de exceion de id ivalido `throw new HttpException('modulo no existe, id inválido',HttpStatus.NOT_FOUND)` y el codigo de http `HttpStatus.NOT_FOUND`.

```ts title="src/apps/modulo/modulo.service.ts"
    async findById(id:string){
        try{
            return await this.RolAppModel.findById(id)
        }catch(e){
            throw new HttpException('el modulo no se encuntra registrado, id invalido',HttpStatus.NOT_FOUND)
        }
    }
```

### Funcion create(body:CreateModuloDto)

Esta funcion retorna y registra los datos a la base de datos, primeramente se verifica `if(find)` que no haya datos con el mismo nombre en la base de datos. caso contrario retorna el mensaje de datos existentes `throw new HttpException('la modulo ya existe',409)`. tambien verifica existentes en la base de datos pero con estado eliminado. si la afirmacion es positiva solo se cambia el estado de eliminacion `find.isDeleted = false`. Si en caso no hay datos similares en la base de datos se registra los datos en la base de datos `return await this.moduloModel.create(body)`.

```ts title="src/apps/modulo/modulo.service.ts"
    async create(body:CreateRolAppDto){
        const {titulo} = body
        const {apps} = body
        const {permisos} = body
        const existePer = this.virificar(permisos)
        if(!(await existePer).valueOf()){
            throw new HttpException('permiso no encontrado', 404)
        }
        try{
            await this.appModel.findById(apps) 
        }catch(e)
        {
            throw new HttpException('aplicacion no encontrado',HttpStatus.NOT_FOUND)
        }
        const find = await this.RolAppModel.findOne({titulo: titulo.toLocaleLowerCase()})
        if(find){
            if(find.isDeleted){
                find.isDeleted = false
                return find.save()
            }
            throw new HttpException('el rol ya existe en app',409)
        }
        return await this.RolAppModel.create(body)

    }
```

### Funcion update(id:string, body:CreateModuloDto)

Esta funcion actualiza y retorna los datos, vereficando que no existan datos semiliares `if(find)`. en caso positivo retorna el mensaje de excepcion `throw new HttpException('la modulo ya existe',409)`, la funcion `toLocaleLowerCase()` permite convertir a minusculas los datos, en este caso el titulo del modulo. la sentencia `const { titulo } = body` extrae el titulo de la modulo. La sentencia `await this.findById(id)` ejecuta a la funcion `findById(id)`, para verificar las exceciones de id.

```ts title="src/apps/modulo/modulo.service.ts"
    async update(id:string, body:CreateRolAppDto){
        await this.findById(id)
        const {titulo} = body
        const find = await this.RolAppModel.findOne({titulo: titulo.toLocaleLowerCase()})
        if(find){
            throw new HttpException('el permiso ya esta registrado',409)
        }
        return await this.RolAppModel.findByIdAndUpdate(id,body,{new:true})
    }
```

### Funcion delete(id:string)

Esta funcion permite eliminar datos del modulo, sin embargo no elimina datos de la base de datos solo cambia el estado de eliminacion del modulo `deleted.isDeleted = true`. La sentencia `await this.findById(id)` ejecuta a la funcion `findById(id)`, para verificar las excepciones de id.

```ts title="src/apps/modulo/modulo.service.ts"
    async delete(id:string){
        await this.findById(id)
        const deleted = await this.RolAppModel.findOne({_id: id})
        if(deleted){
            deleted.isDeleted = true
            return deleted.save()
        }
    }
```

## Schema

Es la esquema que interactua con la base de datos, la sentencia `export type ModuloDocument = HydratedDocument<Modulo>` está exportando la esquema del modelo. con el decorador `@Schema()` indica la esquema que se va construir, el decorador `@Prop()` endica la declaración de variables.

```ts title="src/apps/modulo/schema/modulo.schema.ts"
import { Prop, Schema, SchemaFactory } from "@nestjs/mongoose";
import { HydratedDocument} from "mongoose";
import { v4 as uuidv4 } from 'uuid';

export type RolAppDocument = HydratedDocument<RolApp>

@Schema()
export class RolApp{

    @Prop({ type:String, default: () => uuidv4() })
    uuid: string

    @Prop({required:true})
    titulo:string
    @Prop()
    app: string
    @Prop({required:true})
    permisos:[]
    @Prop({default:false})
    isDeleted?:boolean
}
export const RolAppSchema = SchemaFactory.createForClass(RolApp)
```

## Dto

La clase `CreateModuloDto` permite crear como un respaldo de esquema, tambien es donde se pueden definir las variables para la base de datos y los decoradores a la nesecidad. el decaroador `@ApiProperty()` permite mostrar los campos de insercion por swagger, el decorador `@IsNotEmpty()` hará que no se inserte campos vacíos y mostrará el mensaje que esta configurado. El decorador `@Matches(/^[a-zA-Z]{3,64}$/)` permita insertar caracteres que se ah espeficicado en el decorador.

```jsx title="src/apps/modulo/dto/create-modulo.dto.ts"
import { ApiProperty } from "@nestjs/swagger";
import { IsNotEmpty, Matches} from "class-validator";

export class CreateRolAppDto {

    @ApiProperty()
    @IsNotEmpty({message:'el campo titulo es requerido'})
    @Matches(/^[a-zA-Z0-9' '_-]{3,64}$/,{message:'el campo titulo debe contener solo caracteres' +
    ' y debe ser como minima 3 y maxima 64 caracteres '})
    titulo: string
    @ApiProperty()
    apps:string
    @ApiProperty()
    permisos:[]

    isDeleted?:boolean
}
```
