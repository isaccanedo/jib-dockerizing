## Jib

# 1. Visão Geral
Neste tutorial, daremos uma olhada no Jib e como ele simplifica a conteinerização de aplicativos Java.

Pegaremos um aplicativo Spring Boot simples e construiremos sua imagem Docker usando Jib. E então publicaremos a imagem em um registro remoto.

E certifique-se também de consultar nosso tutorial sobre encaixe de aplicativos Spring Boot usando dockerfile e a ferramenta docker.

# 2. Introdução ao Jib
Jib é uma ferramenta Java de código aberto mantida pelo Google para a construção de imagens Docker de aplicativos Java. Isso simplifica a conteinerização, já que, com ela, não precisamos escrever um dockerfile.

E, na verdade, nem mesmo precisamos ter o docker instalado para criar e publicar as imagens do docker.

O Google publica o Jib como um plug-in Maven e Gradle. Isso é bom porque significa que o Jib pegará todas as mudanças que fizermos em nosso aplicativo cada vez que construirmos. Isso nos economiza comandos docker build / push separados e simplifica sua adição a um pipeline de CI.

Existem algumas outras ferramentas por aí também, como os plug-ins docker-maven-plugin e dockerfile-maven do Spotify, embora o primeiro esteja obsoleto e o último exija um dockerfile.

# 3. Um aplicativo de saudação simples
Vamos pegar um aplicativo simples de inicialização rápida e encaixá-lo usando o Jib. Ele exporá um ponto de extremidade GET simples:

```
http://localhost:8080/greeting
```

O que podemos fazer simplesmente com um controlador Spring MVC:

```
@RestController
public class GreetingController {

    private static final String template = "Hello Docker, %s!";
    private final AtomicLong counter = new AtomicLong();

    @GetMapping("/greeting")
    public Greeting greeting(@RequestParam(value="name", 
        defaultValue="World") String name) {
        
        return new Greeting(counter.incrementAndGet(),
          String.format(template, name));
    }
}
```

# 4. Preparando a implantação

Também precisaremos nos configurar localmente para autenticar com o repositório Docker que queremos implantar.

Para este exemplo, forneceremos nossas credenciais DockerHub para .m2/settings.xml:

```
<servers>
    <server>
        <id>registry.hub.docker.com</id>
        <username><DockerHub Username></username>
        <password><DockerHub Password></password>
    </server>
</servers>
```

Existem outras maneiras de fornecer as credenciais também. A maneira recomendada pelo Google é usar ferramentas auxiliares, que podem armazenar as credenciais em um formato criptografado no sistema de arquivos. Neste exemplo, poderíamos ter usado docker-credential-helpers em vez de armazenar credenciais de texto simples em settings.xml, que é muito mais seguro, embora simplesmente fora do escopo deste tutorial.

# 5. Implantando no Docker Hub com Jib
Agora, podemos usar jib-maven-plugin, ou o equivalente do Gradle, para armazenar nosso aplicativo com um comando simples:

```
mvn compile com.google.cloud.tools:jib-maven-plugin:2.5.0:build -Dimage=$IMAGE_PATH
```

onde IMAGE_PATH é o caminho de destino no registro do contêiner.

Por exemplo, para fazer upload da imagem isaccanedojib / spring-jib-app para DockerHub, faríamos:

```
export IMAGE_PATH=registry.hub.docker.com/isaccanedojib/spring-jib-app
```

E é isso! Isso construirá a imagem do docker de nosso aplicativo e a enviará para o DockerHub.

Podemos, é claro, fazer upload da imagem para o Google Container Registry ou Amazon Elastic Container Registry de maneira semelhante.

# 6. Simplificando o Comando Maven
Além disso, podemos encurtar nosso comando inicial configurando o plugin em nosso pom, como qualquer outro plugin maven.

```
<project>
    ...
    <build>
        <plugins>
            ...
            <plugin>
                <groupId>com.google.cloud.tools</groupId>
                <artifactId>jib-maven-plugin</artifactId>
                <version>2.5.0</version>
                <configuration>
                    <to>
                        <image>${image.path}</image>
                    </to>
                </configuration>
            </plugin>
            ...
        </plugins>
    </build>
    ...
</project>
```

Com essa mudança, podemos simplificar nosso comando maven:

```
mvn compile jib:build
```

# 7. Personalização de aspectos do Docker
Por padrão, Jib faz várias suposições razoáveis sobre o que queremos, como FROM e ENTRYPOINT.

Vamos fazer algumas alterações em nosso aplicativo que são mais específicas para nossas necessidades.

Primeiro, o Spring Boot expõe a porta 8080 por padrão.

Mas, digamos, queremos fazer nosso aplicativo rodar na porta 8082 e torná-lo exposta por meio de um contêiner.

Claro, faremos as alterações apropriadas no Boot. E, depois disso, podemos usar o Jib para torná-lo exposta na imagem:

```
<configuration>
    ...
    <container>
        <ports>
            <port>8082</port>
        </ports>
    </container>
</configuration>
```

Ou digamos que precisamos de um FROM diferente. Por padrão, o Jib usa a imagem java sem distro.

Se quisermos executar nosso aplicativo em uma imagem base diferente, como alpine-java, podemos configurá-lo de maneira semelhante:

```
<configuration>
    ...
    <from>                           
        <image>openjdk:alpine</image>
    </from>
    ...
</configuration>
```

Configuramos tags, volumes e várias outras diretivas do Docker da mesma maneira.

# 8. Customizando Aspectos Java

E, por associação, o Jib também oferece suporte a várias configurações de tempo de execução Java:

- jvmFlags é para indicar quais sinalizadores de inicialização devem ser passados para a JVM;
- mainClass serve para indicar a classe principal, que Jib tentará inferir automaticamente por padrão;
- args é onde especificamos os argumentos do programa passados para o método principal.

Claro, certifique-se de verificar a documentação do Jib para ver todas as propriedades de configuração disponíveis.

# 9. Conclusão
Neste tutorial, vimos como construir e publicar imagens do docker usando o Jib do Google, incluindo como acessar as diretivas do docker e as configurações de tempo de execução Java por meio do Maven.