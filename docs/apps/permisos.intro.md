# Configuraciones para los Permisos

## Controller

### Creacion de Controller

Realizamos la creacion de controlador en la ruta `src/apps/permisos/permisos.controller.ts`, el decorador `@ApiTags('permiso')` permite mostrar los endpoints en swagger, el decorador `@controller('permisos')` permite crear el controlador. finalmente se exporta el controlador.

```ts title="src/apps/permisos/permisos.controller.ts"
import { Body, Controller, Delete, Get, HttpException, HttpStatus, Param, Post, Put, UsePipes, ValidationPipe } from "@nestjs/common";
import { ApiTags } from "@nestjs/swagger";
import { CreatePermisoAppDto } from "./dto/create-permisos.dto";
import { PermisosAppService } from "./permisosapp.service";


@ApiTags('Permiso-App')
@Controller('Permiso-App')
export class PermisosAppController {
    constructor(private readonly permisosAppService:PermisosAppService){}
  //resto del codigo como @Get(), @Post(), @Put(), @Delete(), etc
  }
```

### Metodo Get

El decorador `@Get()` indica que se va ejecutarme un endpoint get, la funcion retorna los datos retuperados de la base de datos asia el cliente.
![appasas-get](/img/perapp-get.png) 
```ts title="src/apps/permisos/permisos.controller.ts"
    @Get()
    async findAll(){
        return await this.permisosAppService.findAll();
    }
```

### Metodo Get por ID

Esta funcion permitira el filtro de una aplicacion desde su id, returna los datos recuperados de la base de datos hacia el cliente, si embargo no retorna a aquellos aplicaciones que estan con estados elimiando `if(permiso.isDeleted)`, simplemente devuelve el error de mensaje `throw new HttpException('no se encuentra el permiso',HttpStatus.NOT_FOUND)`.
![appasas-get](/img/perapp-get-i.png) 
```ts title="src/apps/permisos/permisos.controller.ts"
    @Get(':id')
    async getById(@Param('id') id:string){
        const permiso = await this.permisosAppService.findByid(id)
        if(permiso.isDeleted){
            throw new HttpException('no se encuentra el permiso',HttpStatus.NOT_FOUND)
        }
        return permiso
    }
```

### Metodo Post

Esta funcion retorna y registra los datos a la base de datos insertados por el endpoint post, recive como parametro la configuracion en dto. Los descoradores `@UsePipes(new ValidationPipe())` permite mostras los mensajes de errorres que se configuro en dto.
![appasas-get](/img/perapp-post.png) 
```ts title="src/apps/permisos/permisos.controller.ts"
    @Post()
    @UsePipes(new ValidationPipe())
    async createPermiso(@Body() Body:CreatePermisoAppDto){
        return await this.permisosAppService.create(Body)
    }
```

### Metodo Put
![appasas-get](/img/perapp-put.png) 
Esta funcion returna y permite realizar los cambios de campos del permiso, recive en su paramentro la configuracion realizada en dto. Los descoradores `@UsePipes(new ValidationPipe())` permite mostras los mensajes de errorres que se configuro en dto.

```ts title="src/apps/permisos/permisos.controller.ts"
    @Put('/update:id')
    @UsePipes(new ValidationPipe())
    async updatePermiso(@Param('id') id:string, @Body() Body: CreatePermisoAppDto){
        return await this.permisosAppService.update(id,Body)
    }
```

### Metodo Delete

Esta funcion permite eleminar los datos del permiso, la sentencia `throw new HttpException('modulo eliminada',HttpStatus.OK)` retorna un mensaje de exito de eliminacion, si embargo esta funcion no elimana los datos del permiso en la base de datos, solo cambia el estado de eliminacion.
![appasas-get](/img/perapp-delete.png) 
```ts title="src/apps/permisos/permisos.controller.ts"
    @Delete('/delete:id')
    async deletePermiso(@Param('id') id:string){
        await this.permisosAppService.remove(id)
        throw new HttpException('permiso eliminado',HttpStatus.OK)
    }
```

### Module

En este aparato se crea los modules, los `imports` importa los modelos, el `MongooseModule.forFeature` sirve para importar modelos al moongose, como es este caso el modelo de permisos, con `name:Permisos.name` hace referncia al nombre de la clase module y `schema:PermisosSchema` importa la esquema del modelo donde se esta trabajando. El `contorllers` importa los controladores que sean necesarios, en este caso `PermisosController` esta importando el controlador de permisos. El`PermisosService` importa los servicios que se desea, en este caso `AppsService` esta importando los servicios de los permisos.

```ts title="src/apps/permisos/permisos.module.ts"
import { Module } from "@nestjs/common";
import { MongooseModule } from "@nestjs/mongoose";
import { PermisosApp, PermisosAppSchema } from "./schema/permisosapp.schema";
import { PermisosAppController } from "./permisosapp.controller";
import { PermisosAppService } from "./permisosapp.service";


@Module({
    imports:[MongooseModule.forFeature([
        {
            name:PermisosApp.name,
            schema:PermisosAppSchema
        }
    ]),
],
controllers:[PermisosAppController],
providers:[PermisosAppService]
})
export class PermisoAppMoudle {}
```

### Service

El decorador `@Injectable()` indica la inyeccion de dependencias. En el constructor se esta inyectando el modelo de las modulo obteniendo de la esquema `ModuloDocument`.

```ts title="src/apps/permisos/permisos.service.ts"
import { InjectModel } from "@nestjs/mongoose";
import { Model } from "mongoose";
import { PermisosApp, PermisosAppDocument } from "./schema/permisosapp.schema";
import { HttpException, HttpStatus, Injectable } from "@nestjs/common";
import { CreatePermisoAppDto } from "./dto/create-permisos.dto";

@Injectable()
export class PermisosAppService{
    constructor(
        @InjectModel(PermisosApp.name) private readonly permisoAppModel: Model<PermisosAppDocument>
    ){}
  // El resto del código las funciones de Get Post Put, etc
}
```

### Funcion findAll()

La funcion retorna todos los datos de permisos de la base de datos, si embargo no retorna los permisos que estan con estado eliminado, la ejecucion realiza a traves del modelo `this.permisoModel`.

```ts title="src/apps/permisos/permisos.service.ts"
      async findAll(){
        return await this.permisoAppModel.find({isDeleted:false}).exec()
    }
```

### Funcion findPermiso(id:string)

Esta funcion retorna la informacion de un permiso por el id, pero siempre en cuando no se presente alguna excepcion, por ejemplo puede ver casos en que la id sea incorrecto, si fuera el caso devuelve un error de exceion de id ivalido `throw new HttpException('permiso no resgistrado, id inválido',HttpStatus.NOT_FOUND)` y el codigo de http `HttpStatus.NOT_FOUND`.

```ts title="src/apps/permisos/permisos.service.ts"
    async findByid(id:string){
        try{
            return await this.permisoAppModel.findById(id)
        }catch(e){
            throw new HttpException('permiso no resgistrado, id ivalido',HttpStatus.NOT_FOUND)
        }
    }
```

### Funcion create(Body:CreatePermisoAppDto)

Esta funcion retorna y registra los datos a la base de datos, primeramente se verifica `if(find)` que no haya datos con el mismo nombre en la base de datos. caso contrario retorna el mensaje de datos existentes `throw new HttpException('la modulo ya existe',409)`. tambien verifica existentes en la base de datos pero con estado eliminado. si la afirmacion es positiva solo se cambia el estado de eliminacion `find.isDeleted = false`. Si en caso no hay datos similares en la base de datos se registra los datos en la base de datos `return await this.moduloModel.create(body)`.

```ts title="src/apps/permisos/permisos.service.ts"
    async create(Body:CreatePermisoAppDto){
        const {name} = Body
        const findPer = await this.permisoAppModel.findOne({name: name})
        if(findPer){
            if(findPer.isDeleted){
                findPer.isDeleted = false
                return findPer.save()
            }
            throw new HttpException('el permiso ya existe',409)
        }
        return await this.permisoAppModel.create(Body)
    }
```

### Funcion update (id:string, body:CreatePermisoDto)

Esta funcion actualiza y retorna los datos, vereficando que no existan datos semiliares `if(findFer)`. en caso positivo retorna el mensaje de excepcion `throw new HttpException('el permiso ya esta registrado',409)`, la funcion `toLocaleLowerCase()` permite convertir a minusculas los datos, en este caso el permiso de permisos. la sentencia `const {permiso} = body` extrae el el permiso de permisos. La sentencia `await this.findPermiso(id)` ejecuta a la funcion `findPermiso(id)`, para verificar las exceciones de id.

```ts title="src/apps/permisos/permisos.service.ts"
    async update (id:string, body:CreatePermisoAppDto){
        await this.findByid(id)
        const {name} = body
        const findFer = await this.permisoAppModel.findOne({name: name})
        if(findFer){
            throw new HttpException('el permiso ya esta registrado',409)
        }
        return await this.permisoAppModel.findByIdAndUpdate(id,body,{new:true})
    }
```

### Funcion remove(id:string)

Esta funcion permite eliminar datos de permisos, sin embargo no elimina datos de la base de datos solo cambia el estado de eliminacion del permisos`deleted.isDeleted = true`. La sentencia `await this.findPermiso(id)` ejecuta a la funcion `findPermiso(id)`, para verificar las excepciones de id.

```ts title="src/apps/permisos/permisos.service.ts"
    async remove(id:string){
        await this.findByid(id)
        const deleted = await this.permisoAppModel.findOne({_id: id})
        if(deleted){
            deleted.isDeleted = true
            return deleted.save()
        }
    }
```

## Schema

Es la esquema que interactua con la base de datos, la sentencia `export type PermisosDocument = HydratedDocument<Permisos>` está exportando la esquema del modelo. con el decorador `@Schema()` indica la esquema que se va construir, el decorador `@Prop()` endica la declaración de variables.

```ts title="src/apps/permisos/schema/permisos.schema.ts"
import { Prop, Schema, SchemaFactory } from "@nestjs/mongoose";
import { HydratedDocument } from "mongoose";
import { v4 as uuidv4 } from 'uuid';

export type PermisosAppDocument = HydratedDocument<PermisosApp>

@Schema()
export class PermisosApp {
    @Prop({type: String, default:()=> uuidv4()})
    uuid:string

    @Prop({required:true})
    name:string
    
    @Prop({default:false})
    isDeleted?:boolean
}
export const  PermisosAppSchema = SchemaFactory.createForClass(PermisosApp)
```

## Dto

La clase `CreateModuloDto` permite crear como un respaldo de esquema, tambien es donde se pueden definir las variables para la base de datos y los decoradores a la nesecidad. el decaroador `@ApiProperty()` permite mostrar los campos de insercion por swagger, el decorador `@IsNotEmpty()` hará que no se inserte campos vacíos y mostrará el mensaje que esta configurado. El decorador `@Matches(/^[a-zA-Z]{3,64}$/)` permita insertar caracteres que se ah espeficicado en el decorador.

```ts title="src/apps/permisos/dto/create-permisos.dto.ts"
import { ApiProperty } from "@nestjs/swagger";
import { IsNotEmpty, Matches } from "class-validator";

export class CreatePermisoAppDto{
    @ApiProperty()
    @IsNotEmpty({message:'el campo name es requerido'})
    @Matches(/^[a-zA-Z' '_-]{3,64}$/,{message:'el campo name debe contener solo caracteres' +
    ' y debe ser como minima 3 y maxima 64 caracteres '})
    name:string
    
    isDeleted?:boolean
}
```
