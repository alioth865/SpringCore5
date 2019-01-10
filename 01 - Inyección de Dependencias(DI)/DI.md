# Inyección de Dependecias(DI)

## Como trabaja Spring:

![Como trabaja Spring](https://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/images/container-magic.png)
#### Que es una inyección de dependencia y cuáles son sus ventajas:
La inyección de dependencias es el proceso mediante el framework de Spring establece las relaciones entre diferentes partes de la aplicación. O sea, en lugar de que cada clase tenga que instanciar los objetos que necesite, es Spring quien se encarga de inyectar(crear) estos objetos.
Cuando usamos el framework de Spring para desarrollar en Java, algunas de las ventajas son:
- Reduce el acoplamiento entre distintas partes de la aplicación
- Incrementa la cohesión de las partes de la aplicación
- Hace más fácil la testeabilidad
- El diseño de la aplicación es mucho más entendible
- Incrementa la reusabilidad
- Incrementa el mantenimiento
- Reduce el código repetitivo

**Un ejemplo practico**
Supongamos que tenemos las siguientes clases de negocio:

```java
public class TransferServiceImpl implements TranferService {
        public TransferServiceImpl(AccountRepository ar) {
                this.accountRepository = ar;
        }
}
```
```java
public class JdbcAccountRepository implements AccountRepository {
        public JdbcAccountRepository(DataSource ds) {
                this.dataSource = ds;
        }
}
```
Esta sería la configuración que le vamos a decir a Spring que haga para que haga el ensamblaje

```java
@Configuration
public class ApplicationConfig{
       @Bean 
       public TransferService transferService() {
              return new TransferServiceImpl( accountRepository() );
       }
       
       @Bean 
        public AccountRepository accountRepository(){
              return new JdbcAccountRepository( dataSource() );
       }
       
       @Bean
       public DataSource dataSource() {
              BasicDataSource dataSource = new BasicDataSource();
              dataSource.setDriverCalssName("org.postgresql.Driver");
              dataSource.setUrl("jdbc:postgresql://localhost/transfer");
              dataSource.setUsername("user");
              dataSource.setPassword("secret45")
              return dataSource;
       }
}
```

Cómo se crea y se usa el aplication context:
```java
ApplicationContext context = SpringApplication.run(ApplicationConfig.class);

//Hay otras formas de obtener el Bean, pero esta suele ser la mas recomendada
//ya que se busca por el BeanID y no hay que castear ya que se le indica el nombre
//de la clase
TransferSercive service = context.getBean( "transferService", TransferService.class);

service.transfer( new MonetaryAmout("300.00"), "CUENTA A", "CUENTA B" );
```
**Breve resumen:**
- Spring separa la configuración de la aplicación de los objetos de aplicación.
- Spring monitorea tus objetos de aplicacioón
-- Creandolos en el orden correcto
-- Garantizando que está inicializado antes de su uso
- Cada bean debe recibir un id/name unico
-- Debe reflejar el servicio o role que pproporciona al cliente
-- El nombre no debe contener detalles de implementación

**Como crea multiples ficheros de configuracion**
Las clases de configuracion @Configuratio puede ser muy largas, asi que se recomienda separar por responsabilidades, y luego importarlos en un único config.

```java
@Configuration
@Import({ApplicationConfig.class, WebLogic.class})
public class infrastructureConfig {
  ... 
}
@Configuration
public class ApplicationConfig {
  ... 
}
@Configuration
public class WebLogic {
  ... 
}
```
Un ejemplo de separación de responsabilidades  aplicados al ejemplo ApplicationConfig es separarlo en dos archivos de configuracion, uno definirá los Beans de aplicacion y en otro los bean de Intraestructura.

```java
@Configuration
public class ApplicationConfig{
       @Bean 
       public TransferService transferService() {
              return new TransferServiceImpl( accountRepository() );
       }
       
       @Bean 
        public AccountRepository accountRepository(){
              return new JdbcAccountRepository( dataSource ); // Ya no se llama dataSource(),
              // simplemente dataSource, y Spring sabe a que se refiere.
       }
}
```

```java
@Configuration
@Import(ApplicationConfig.class)
public class InfrastructureConfig{
       @Bean
       public DataSource dataSource() {
              BasicDataSource dataSource = new BasicDataSource();
              dataSource.setDriverCalssName("org.postgresql.Driver");
              dataSource.setUrl("jdbc:postgresql://localhost/transfer");
              dataSource.setUsername("user");
              dataSource.setPassword("secret45")
              return dataSource;
       }
}
```

```java
//Luego para usarlo solo se necesita hacer esto. 
ApplicationContext ctx = SpringApplication.run(InfrastructureConfig.class)
```

**Ambitos de los beans**

- default
-- Por defecto es singleton 
```java
AccountService s1 = context.getBean("accountService", AccountService.class);
AccountService s2 = context.getBean("accountService", AccountService.class);
assert s1 == s2; //True - Es el mimso objeto.
```
- prototype
-- Una nueva instancia es creada cada vez que se referencia al bean.
```java
@Bean
@Scope( "prototype")
publc AccountService accountService() {
 ...
}
```
```java
@Bean
@Scope( "prototype")
publc AccountService accountService() {
 .....
}
```
```java
AccountService s1 = context.getBean("accountService", AccountService.class);
AccountService s2 = context.getBean("accountService", AccountService.class);
assert s1 == s2; //Falso - No es el mimso objeto.
```

**Ambitos mas comunes**

| Scope | Tiempo de vida |
|:--|:--|
| singleton | Es usada una simple instancia |
| prototype | una nueva instancia es creado cada vez que el bean es referenciado |
| session | Una nueva instancia es creada por sesion de usuario (Solo entorno web) |
| request | Una nueva instancia es creada por request (Solo entornos web) |

