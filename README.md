# PicPay Design Patterns em PHP 8
Repo do código exemplo da Talk Design Patterns em PHP 8 por Leonardo Tumadjian

Simples implementação:

```php
// Implementação de persistencia genérica em banco:

// Usabilidade básica sem implementação
$dataMapper = new Persistence();

$obj1 = new StdClass();
$obj2 = new StdClass();

$dataMapper->persist($obj1);
$dataMapper->delete($obj2);

$dataMapper->flush();
```

Implementação com Design Patterns (Client):

```php
// Usabilidade com Design Patterns
// FactoryMethod > Proxy > TemplateMethod(interface) > Strategy > Decorator

// 1. Escolhemos a decoração via $config['persistenceClass']
// 2. Usamos o factory para criar a instância Proxy
// 3. Proxy recebe PersistenceStrategy com sua classe de tipo de persistencia
$dataMapper = PersistenceFactory::create($config['persistenceClass']);

$obj1 = new StdClass();
$obj2 = new StdClass();

// 4. O método invocado depende da escolha da decoração
// em $config
// ex: "MongoDB"
$dataMapper->persist($obj1);
$dataMapper->delete($obj2);

$dataMapper->flush();
```

Interface PersistenceTemplate (pode-se usar interface segregation)
```php
interface PersistenceTemplate
{
    public function persist(object $object): void;
    public function delete(object $object): void;
    public function flush(): bool;
}
```

Strategy implementado
```php
// Implementação da classe Strategy
class PersistenceStrategy implements PersistenceTemplate
{
    // property promotion PHP 8
    // Strategy define objetos do tipo PersistenceTemplate
    // Decorado por composição
    public function __construct(
        protected PersistenceTemplate $persistence
    ) { }

    public function persist(object $object): void
    {
        $this->persistence->persist($object);
    }

    public function delete(object $object): void
    {
        $this->persistence->delete($object);
    }

    public function flush(): bool
    {
        return $this->persistence->flush();
    }
}
```

Implementação de persistencia em MongoDB(classe exclusiva) pode ter segregação de interfaces (ex: PersistenceTemplate, FilterTemplate)
```php
// Implementação da classe Strategy
class MongoDB implements PersistenceTemplate
{    
    public function persist(object $object): void
    {
        // implementação de persistencia e update Mongo
    }

    public function delete(object $object): void
    {
        // implementação de delete Mongo
    }

    public function flush(): bool
    {
        return true;
    }
}
```

Criando o Factory Method que instancia o objeto Proxy de PersistenceStrategy
obs: o Proxy é opcional
```php
// Factory Method pode ser trocado por 
// Dependency Injection Container em Controller ou UseCase
class PersistenceFactory
{
    public static function create(string $class)
    {
        if (false === class_exists($class)) {
            throw new \Exception('Class do not exist');
        }
        // Trabalhando com o Proxy implementado, melhor aplicado em Dependency Injection Container
        return new PersistenceProxy(new PersistenceStrategy(new $class));
    }
}
```
Implementação do Proxy Cache como exemplo, o mesmo poderia segmentar
Cache X ElasticSearch
```php
// Persistence Proxy
// Uma classe para receber o strategy e adicionar mais
// lógica ao Strategy de forma global a todos objetos
class PersistenceProxy implements PersistenceTemplate
{
    // Proxy para lógica pré e post executions
    public function __construct(
        protected PersistenceStrategy $persistenceStrategy
    ) { }

    public function persist(object $object): void
    {
        // Crie lógica aqui antes de persistir
        // ...
        $this->persistenceStrategy->persist($object);
        // Lógica depois: renova o cache
        // ...
    }

    public function delete(object $object): void
    {
        $this->persistenceStrategy->delete($object);
        // remove elemento do cache / renova cache
    }

    public function flush(): bool
    {
        return $this->persistenceStrategy->flush();
    }
}
```
