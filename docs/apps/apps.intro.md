# Configuraciones para las Aplicaciones

## Controller

El llamado **AppsController** en un controlador que maneja solicitudes HTTP e interactúa con el servicio **AppsService** Las sentencias **import** se utilizan para importar las dependencias y módulos necesarios desde las bibliotecas y archivos, ``@ApiTags("app")`` es un decorador de la librería ``@nestjs/swagger`` que agrega etiquetas a la documentación **Swagger**, ``@Controller("api/apps")`` establece la ruta base para este controlador en **/api/apps**, ``@UseGuards(RolesGuard)`` aplica el guardia **RolesGuard**, que se encarga del control de acceso basado en roles, ``@ApiBearerAuth()`` es un decorador de **nestjs/swagger** que especifica que este controlador requiere autenticación mediante un token. El constructor inicializa el **AppsController** con una instancia de **AppsService**, los métodos del controlador como ``@Get()``, ``@Post()``, ``@Put()``, y ``@Delete()`` son métodos que manejan solicitudes HTTP específicas y delegan la ejecución a los métodos correspondientes en **AppsService** . ``@Body``, ``@Param`` y ``@Res`` son decoradores utilizados para extraer el cuerpo de la solicitud, los parámetros de ruta y los objetos de respuesta,``@Permissions(...)`` es un decorador personalizado que se puede usar para especificar los permisos requeridos para acceder a un controlador o método en particular, ``@ValidationPipe`` y ``@UsePipes`` se utilizan para aplicar la validación definidos en dto.

```ts title="src/apps/apps.controller.ts"
import { Controller, Get, Post, Body, Patch, Param, Delete, UseGuards, Put, Res, ValidationPipe, UsePipes } from '@nestjs/common';
import { AppsService } from './apps.service';
import { CreateAppDto } from './dto/create-system.dto';
import { ApiBearerAuth, ApiTags } from '@nestjs/swagger';
import { RolesGuard } from 'src/auth/guards/roles.guard';
import { Response } from 'express';
import { Permission } from 'src/auth/guards/constants/permission';
import { Permissions } from 'src/auth/guards/decorators/permissions.decorator';

@ApiTags('app')
@Controller('apps')
export class AppsController {
  constructor(private readonly appsService: AppsService) {}
  //resto del codigo como @Get(), @Post(), @Put(), @Delete(), etc
}
```
### Metodo Get
``@Get()`` es un decorador que indica las solicitudes HTTP GET, ``async findAll()`` es una función asincrónica que devuelve una promesa, a tra vez de servivicos, esta funcion retornará todos que devuelve los servicios.
![app-get](/img/app-get.png)
```ts title="src/apps/apps.controller.ts"
  @Get()
  async findAll() {
    return await this.appsService.findAll();
  }
```
### Metodo Get por ID
El método **getAppById()** es un controlador de tipo GET que maneja una solicitud para obtener una aplicación por su ID, ``@Get(':id')`` es un decorador que indica que este método maneja solicitudes HTTP GET con un parámetro de ruta id. ``@Param('id')`` **id: string** es un decorador que extrae el valor del parámetro de ruta id y lo asigna a la variable id como una cadena. En resumen este metodo permite extraer los datos de la base de datos a traves de su id.
![app-get-i](/img/app-get-i.png)
```ts title="src/apps/apps.controller.ts"
  @Get(':id')
  async getAppById(@Param('id') id:string) {
    return await this.appsService.findOne(id);
  }
```

### metodo Post

El método **createApp()** es un controlador de tipo POST, ``@ApiBearerAuth()`` es un decorador que indica que este controlador requiere autenticación con token, ``@Permissions(Permission.CREAR_APP_CEN)`` es un decorador personalizado que especifica los permisos requeridos para acceder a este controlador.``@UseGuards(RolesGuard)`` es un decorador que aplica el guardia **RolesGuard** a este controlador, el cual se encarga del control de acceso basado en roles. ``@UsePipes(new ValidationPipe())`` es un decorador que aplica la validación **ValidationPipe** al objecto **appObject** recibido solicitud. ``@Post()`` es un decorador que indica las solicitudes HTTP POST. En resumen este metodo permite la insertar datos a la abse de datos.
![app-get-i](/img/app-post.png)
```ts title="src/apps/apps.controller.ts"
  @ApiBearerAuth()
  @Permissions(Permission.CREAR_APP_CEN)
  @UseGuards(RolesGuard)
  @UsePipes(new ValidationPipe())
  @Post()
  async createApp (@Body() appObject:CreateAppDto){
    return await this.appsService.createNewApp(appObject)
  }
```

### Metodo Put

``@ApiBearerAuth()`` es un decorador  que se requiere autenticación de token. ``@Permissions(Permission.EDITAR_APP_CEN)`` es un decorador personalizado utilizado para manejar permisos. ``@UseGuards(RolesGuard)`` Es un decorador indica que se debe usar un guardia llamado **RolesGuard** para proteger esta ruta. ``@UsePipes(new ValidationPipe())`` es un decorador de validación que valida el parámetro **appNewObject**, ``@Put('/update-app:id')`` es un decorador que define las solicitudes HTTP PUT en el endpoint **/update-app:id**. Especifica un parámetro dinámico. en resumen este metodo permite actualizar los datos por los servicios **appsService** 
![app-put](/img/app-put.png)
```ts title="src/apps/apps.controller.ts"
  @ApiBearerAuth()
  @Permissions(Permission.EDITAR_APP_CEN)
  @UseGuards(RolesGuard)
  @UsePipes(new ValidationPipe())
  @Put('/update-app:id')
  async updateApp(@Param('id') id:string, @Body() appNewObject: CreateAppDto){
    return await this.appsService.updatedApp(id,appNewObject)
  }
```

### Metodo Delete
``@ApiBearerAuth()``es un decorador de una herramienta de documentación de API, como Swagger, que especifica que el método requiere un token de autorización para la autenticación. ``@Permissions(Permission.ELIMINAR_APP_CEN)`` es un decorador personalizado. ``@UseGuards(RolesGuard)`` es un decorador que indica el uso de un guardia llamado "RolesGuard". ``@Delete('/delete-app:id')`` es un decorador especifica que este método debe manejar solicitudes DELETE HTTP a la ruta **'/delete-app:id'**, **':id'** es un parámetro de ruta que se pasará al método **deleteApp**, en resumen este metodo permite eliminar los datos, sin embargo no elimina de la abse de datos solo cmabia el estado. 
![app-delete](/img/app-delete.png)
```ts title="src/apps/apps.controller.ts"
  @ApiBearerAuth()
  @Permissions(Permission.ELIMINAR_APP_CEN)
  @UseGuards(RolesGuard)
  @Delete('/delete-app:id')
  async deleteApp(@Param('id') id:string, @Res() res: Response){
    await this.appsService.removeApp(id)
    throw new HttpException('aplicacion eliminada',HttpStatus.OK)
  }
```

### Module

En este aparato se crea los modules, los **imports** importa los modelos, el **MongooseModule.forFeature** sirve para importar modelos al moongose, como es este caso el modelo de apps, con **name:App.name** hace referncia al nombre de la clase module y **schema:AppSchema** importa la esquema del modelo donde se esta trabajando, como en este caso **ModuloModule** esta importando el module del modulo y **PermisoMoudle** esta importando el modelo de los permisos. El **contorllers** importa los controladores que sean necesarios, en este caso **AppsController** esta importando el controlador de apps. El **providers** importa los servicios que se desea, en este caso **AppsService** esta importando los servicios de apps.

```ts title="src/apps/apps.module.ts"
import { Module } from "@nestjs/common";
import { AppsService } from "./apps.service";
import { AppsController } from "./apps.controller";
import { MongooseModule } from "@nestjs/mongoose";
import { App, AppSchema } from "./schema/apps.schema";
import { User, UserSchema } from "src/user/schema/user.schema";
import {
  Permission,
  PermissionSchema,
} from "src/permission/schema/permission.schema";
import { Rol, RolSchema } from "src/rol/schema/rol.schema";
import { ModuloModule } from "./modulo/modulo.module";
import { PermisoMoudle } from "./permisos/permisos.module";

@Module({
  imports: [
    MongooseModule.forFeature([
      { name: App.name, schema: AppSchema },
      { name: User.name, schema: UserSchema },
      { name: Permission.name, schema: PermissionSchema },
      { name: Rol.name, schema: RolSchema },
    ]),
    ModuloModule,
    PermisoMoudle,
  ],
  controllers: [AppsController],
  providers: [AppsService],
})
export class AppsModule {}
```
### Service

El decorador `@Injectable()` indica la inyeccion de dependencias. En el constructor se esta inyectando el modelo de las aplicaciones obteniendo de la esquema **AppDocument**.

```ts title="src/apps/apps.service.ts"
import { HttpException, Injectable, NotFoundException } from '@nestjs/common';
import { App, AppDocument } from './schema/apps.schema';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { User, UserDocument } from 'src/user/schema/user.schema';
import { CreateAppDto } from './dto/create-system.dto';

@Injectable()
export class AppsService {
  
  constructor(
    @InjectModel(App.name) private readonly appModel: Model<AppDocument>,
  @InjectModel(User.name) private readonly userModel: Model<UserDocument>
  ){}
  // El resto del código las funciones de Get Post Put, etc
}
```

### Seeders

Los seeder son datos que se almacenan en la base de datos que podrían ser por defecto o ejecutando algun comando, en este caso se esta pretendiendo almacenar los datos a la base de datos, verificando (`if (count > 0) return`)siempre cuando la base de datos este vacío a traves del modelo. La funcion `estimatedDocumentCount()` es para recuperar la cantidad de datos que existen en la base de datos.

```ts title="src/apps/apps.service.ts"
async setAppsDefault() {
    const count = await this.appModel.estimatedDocumentCount();
    if (count > 0) return;
    const values = await Promise.all([
      this.appModel.create({ name: 'personal', expiresIn:'4h'}),
      this.appModel.create({ name: 'activo', expiresIn:'4h' }),
      this.appModel.create({ name: 'central', expiresIn:'6h' }),
      this.appModel.create({ name: 'gestion-documental', expiresIn:'4h' }),
      this.appModel.create({ name: 'biblioteca', expiresIn:'4h' })
    ]);
    return values;
  }
```

Si se quiere que se inserte estes datos por defecto se tiene que dirigirse a main del sistema principal

```ts title="src/main.ts"
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { AppsService } from "./apps/apps.service";

async function bootstrap() {
  const app = await NestFactory.create(AppModule, { cors: true });

  const apps = app.get(AppsService);
  apps.seederApp();

  await app.listen(3000);
}
bootstrap();
```

### Funcion findAll()

Esta funcion retorna todos los datos de aplicaciones de la base de datos, si embargo no retorna las aplicaciones que estan con estado eliminado, la ejecucion se realiza por la eschema **appModel**.

```ts title="src/apps/apps.service.ts"
  async findAll() {
    return await this.appModel.find({ isDeleted: false }).exec();
  }
```

### Funcion findOne(id:string)

Esta funcion retorna la informacion de una aplicacion por el id, verificando que el id sea valido y **throw new HttpException('aplicacion no existe, id inválido',HttpStatus.NOT_FOUND)** genera un error y devuelve sus datos, **HttpStatus.NOT_FOUND** es manejador de estados de nestjs.

```ts title="src/apps/apps.service.ts"
  async findOne(id:string){
    const findApp = await this.appModel.findOne({uuid:id})
    if(!findApp){
      throw new HttpException('aplicacion no encontrada',404)
    }
    if(findApp.isDeleted == true){
      throw new HttpException('aplicacion eliminada',404)
    }
    return findApp
  }
```

### Funcion createNewApp(appObject:CreateAppDto)

Esta funcion retorna y registra los datos a la base de datos, verificando que los datos insertados no existan en la base de datos **if(findApp)**, sis esto se cumple retorna el error. de la misma manera verifica si hay datos con estado eliminado, si se cumple esta condición mantiene los datos, solo se cambia el estado de eliminacion.

```ts title="src/apps/apps.service.ts"
  async createNewApp(appObject:CreateAppDto){
    const { name } = appObject

    const findApp = await this.appModel.findOne({ name: name.toLocaleLowerCase() })

    if(findApp){
      throw new HttpException('la aplicacion ya existe',409)
    }

    return await this.appModel.create(appObject);
  }
```

### Funcion updatedApp (id:string, appNewObject:CreateAppDto)

Esta funcion actualiza y retorna los datos, vereficando datos existentes en la base de datos **if(findApp)**. si esto se cumple retorna el mensaje de error, la funcion **toLocaleLowerCase()** permite convertir a minusculas los datos.la sentencia findOne(id), llama a la funcion findOne(i), para vefificar que el id intruducido sea lo correcto.

```ts title="src/apps/apps.service.ts"
  async updatedApp (id:string, appNewObject:CreateAppDto){
    return await this.appModel.findByIdAndUpdate(id, appNewObject,{new:true});
  }
```

### Funcion removeApp(id : string)

Esta funcion permite eliminar datos de la aplicacion, sin embargo no elimina datos de la base de datos solo cambia el estado de eliminacion de la aplicacion **deleteApp.isDeleted = true**. La sentencia **await this.findOne(id)** ejecuta a la funcion **findOne(id)**, para verificar que el id intruducido sea correcto.

```ts title="src/apps/apps.service.ts"
  async removeApp(id : string){
    const deleteApp = await this.appModel.findOne({ _id: id })
    if (!deleteApp) {
      throw new HttpException('la aplicacion no existe',409)
    }
    deleteApp.isDeleted = true
    return deleteApp.save()
  }
```

## Schema
La schema permite regitrar y declarar las variables con sus respectivos tipeados para la base de datos, la sentencia `export type AppDocument = HydratedDocument<App>` está exportando la esquema declarado, el decorador `@Schema()` indica la esquema que se va construir, el decorador `@Prop()` endica la declaración de variables. para insercion de datos por defecto se utiliza el decorador:`@Prop({ type:String, default: () => uuidv4() })`, la variable uuid permite crear un id a parte del id que se crea por defecto en cada insercion de datos y finalmete se construye la esquema con esta sentencia **SchemaFactory.createForClass(App)** .

```ts title="src/apps/schema/apps.schema.ts"
import { Schema, SchemaFactory, Prop } from "@nestjs/mongoose";
import mongoose, { HydratedDocument } from "mongoose";
import { Rol } from "src/rol/schema/rol.schema";
import { v4 as uuidv4 } from "uuid";

export type AppDocument = HydratedDocument<App>;

@Schema()
export class App {
  @Prop({ type: String, default: () => uuidv4() })
  uuid: string;

  @Prop({ required: true })
  name: string;

  @Prop({ required: true })
  expiresIn: string;

  @Prop({ default: false })
  isDeleted?: boolean;
}
export const AppSchema = SchemaFactory.createForClass(App);
```

## Dto
El Dto Data Transfer Object que permite transferir datos entre las capas de la aplicacion, esto validará la esturctura de datos y mapeará los datos recividos. La clase **CreateAppDto** es porn donde comunicara el controlador. El decaroador `@ApiProperty()` es un apropiedad de la librería **swagger** que permitira insertar datos pos los endpoits, el decorador `@IsNotEmpty()` es una prppieda de la librería **classs-validator** que verifica la no insercion de campos vacíos. El decorador `@Matches(/^[a-zA-Z]{3,64}$/)` es una propiedad de **class-validator** .

```ts title="src/apps/dto/create-apps.dto.ts"
import { ApiProperty } from "@nestjs/swagger";
import { IsArray, IsNotEmpty, IsString, Matches } from "class-validator";
export class CreateAppDto {

  @ApiProperty()
  @IsNotEmpty({message:'el campo name es requerido'})
  @Matches(/^[a-zA-Z]{3,64}$/,{message:'el campo name debe contener solo caracteres' +
  ' y debe ser como minima 3 y maxima 64 caracteres '})
  name:string;

  @ApiProperty()
  @IsString()
  expiresIn: string;

  isDeleted?: boolean
}
```
