## Recipe to replace Java records with classes

This is an [OpenRewrite](https://docs.openrewrite.org/) recipe to transform Java records to classes.

For instance the following record:

```java
public record Vehicle(String model, int power) {
}
```

can be transformed to the following class:

```java
public final class Vehicle {
    private final String model;
    private final int power;

    public Vehicle(String model, int power) {
        this.model = model;
        this.power = power;
    }

    public String model() {
        return model;
    }

    public int power() {
        return power;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }
        Vehicle other = (Vehicle) obj;
        return Objects.equals(model, other.model) && power == other.power;
    }

    @Override
    public int hashCode() {
        return Objects.hash(model, power);
    }

    @Override
    public String toString() {
        return "Vehicle[model=" + model + ", power=" + power + "]";
    }
}
```

Other examples of code transformation are available in [tests](src/test/java/com/advalange/coderewriter/ReplaceRecordWithClassTest.java).

## Local Publishing for Testing

Before you publish your recipe module to an artifact repository, you may want to try it out locally.
To do this on the command line, run `./gradlew publishToMavenLocal` (or equivalently `./gradlew pTML`).
This will publish to your local maven repository, typically under `~/.m2/repository`.

Replace the groupId, artifactId, recipe name, and version in the below snippets with the ones that correspond to your recipe.

In a Maven project's pom.xml, make your recipe module a plugin dependency:
```xml
<project>
    <build>
        <plugins>
            <plugin>
                <groupId>org.openrewrite.maven</groupId>
                <artifactId>rewrite-maven-plugin</artifactId>
                <version>5.3.2</version>
                <configuration>
                    <activeRecipes>
                        <recipe>com.advalange.coderewriter.ReplaceRecordWithClass</recipe>
                    </activeRecipes>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>com.advalange.coderewriter</groupId>
                        <artifactId>rewrite-recipe-record-to-class</artifactId>
                        <version>0.1.0-SNAPSHOT</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
</project>
```

Unlike Maven, Gradle must be explicitly configured to resolve dependencies from maven local.
The root project of your gradle build, make your recipe module a dependency of the `rewrite` configuration:

```groovy
plugins {
    id("java")
    id("org.openrewrite.rewrite") version("6.1.19")
}

repositories {
    mavenLocal()
    mavenCentral()
}

dependencies {
    rewrite("com.advalange.coderewriter:rewrite-recipe-record-to-class:0.1.0-SNAPSHOT")
}

rewrite {
    activeRecipe("com.advalange.coderewriter.ReplaceRecordWithClass")
}
```

Now you can run `mvn rewrite:run` or `gradlew rewriteRun` to run your recipe.

## Publishing to Artifact Repositories

This project is configured to publish to Moderne's open artifact repository (via the `publishing` task at the bottom of
the `build.gradle.kts` file). If you want to publish elsewhere, you'll want to update that task.
[app.moderne.io](https://app.moderne.io) can draw recipes from the provided repository, as well as from [Maven Central](https://search.maven.org).

Note:
Running the publish task _will not_ update [app.moderne.io](https://app.moderne.io), as only Moderne employees can
add new recipes. If you want to add your recipe to [app.moderne.io](https://app.moderne.io), please ask the
team in [Slack](https://join.slack.com/t/rewriteoss/shared_invite/zt-nj42n3ea-b~62rIHzb3Vo0E1APKCXEA) or in [Discord](https://discord.gg/xk3ZKrhWAb).

These other docs might also be useful for you depending on where you want to publish the recipe:

* Sonatype's instructions for [publishing to Maven Central](https://maven.apache.org/repository/guide-central-repository-upload.html)
* Gradle's instructions on the [Gradle Publishing Plugin](https://docs.gradle.org/current/userguide/publishing\_maven.html).

### From Github Actions

The `.github` directory contains a Github action that will push a snapshot on every successful build.

Run the release action to publish a release version of a recipe.

### From the command line

To build a snapshot, run `./gradlew snapshot publish` to build a snapshot and publish it to Moderne's open artifact repository for inclusion at [app.moderne.io](https://app.moderne.io).

To build a release, run `./gradlew final publish` to tag a release and publish it to Moderne's open artifact repository for inclusion at [app.moderne.io](https://app.moderne.io).
