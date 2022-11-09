# Migrate Spring Cloud Azure project from Spring Boot 2.7 to 3.0

This project is targeted to demo the migration of a project, which using Spring Cloud Azure libraries and Spring Boot 2.7, to Spring Boot 3.0.

## Prerequisites
- Java version 17
   - Spring Boot 3 requires Java 17
   - Spring Boot 2 supports Java8+
- [An Azure subscription](https://azure.microsoft.com/free/)
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- Maven

## Provision Azure Resources

### Create Azure Storage Account

[Create a storage account](https://learn.microsoft.com/azure/storage/common/storage-account-create?tabs=azure-portal)

### Create Azure EventHubs

Create an event hubs namespace and an event hub, see [Create an event hub using Azure portal](https://learn.microsoft.com/azure/event-hubs/event-hubs-create)

### Create Azure Key Vault

Create a key vault to store other resource credentials, see [Create a key vault using the Azure portal](https://learn.microsoft.com/en-us/azure/key-vault/general/quick-create-portal)

### Configure secrets in Azure Key Vault Secret

The application will read the credentials of the Event Hubs and Storage Account, which store in Secrets of the Key Vault. 
Create below 3 secrets in Key Vault:
- eventhubsconnstring, it stores the connection string of the Event Hubs namespace.
- storageaccountname, it stores the storage account name of the Storage Account.
- storageaccountkey, it stores the access key of the Storage Account.

-- No credential stored in KV, we should promote DAC

## Run and verify with Spring Boot 2 

### Run locally with Maven

Open your Command prompt, run `mvn clean spring-boot:run`.

```shell
mvn clean spring-boot:run
```

Send a request to send a message to Event Hubs.

```shell
curl -X POST http://localhost:8080/message/send -H "Content-Type: application/json" -d "{\"content\":\"demo migration\"}"
```

### Verify this sample

Verify the below logs:

```text
Sending message: GenericMessage [payload=DemoEvent{id='1667872542592', content='demo'}, headers={id=52e066d8-fa00-fb04-6570-a38af8bc2595, timestamp=1667872542594}]
```

```text
Message 'DemoEvent{id='1667872542592', content='demo'}' successfully checked.
```

## Migrate to Spring Boot 3

Upgrade to Spring Boot 3.0.0 and Spring Cloud Azure 6.0.0, we expect some known changes can be done by the migration tool `spring-boot-migrator`, it will enhance the Spring Boot 3 support.

### Use spring-boot-migrator to upgrade

1. Update your environment variable `JAVA_HOME` to refer the Java 17.
2. Download the latest release from https://github.com/spring-projects-experimental/spring-boot-migrator/releases.
3. Make your project versioned by Git, which means committing all the changes before scanning the project.
4. Run the jar file with below command:
   `java -jar spring-boot-migrator.jar`
5. Acquire the recipes with below command:
   `scan azure-spring-boot-27-legacy`
   ![scan-project.png](docs/scan-project.png)

   The applicable recipes will be listed in console, there will be the below 3 types:
   - automated recipe
   - partially automated recipe
   - manual recipe

6. Apply the recipe `boot-autoconfiguration-update` to use the new registration file for registering the auto configuration class.
   `apply boot-autoconfiguration-update`
   ![boot-2.7-3.0-dependency-version-update.png](docs/boot-autoconfiguration-update.png)

   After applying the recipe, a log of `SBM: applied recipe 'boot-autoconfiguration-update'` was committed locally.
 
   Actually, the changes are not committed successfully.
   ![file-changes-but-not-commit.png](docs/file-changes-but-not-commit.png)

7. Apply the recipe `boot-2.7-3.0-dependency-version-update` to upgrade dependencies.
   `apply boot-2.7-3.0-dependency-version-update`
   ![boot-2.7-3.0-dependency-version-update.png](docs/boot-2.7-3.0-dependency-version-update.png)

    After applying the recipe, a log of `SBM: applied recipe 'boot-2.7-3.0-dependency-version-update'` was committed locally.

   Actually, partially changes are committed successfully, not all.
   ![partially-changes-committed.png](docs/partially-changes-committed.png)

    Note: The recipe `boot-2.7-3.0-dependency-version-update` will include the recipe `boot-autoconfiguration-update` migration, so we can use this recipe directly.

8. Apply the recipe `boot-2.7-3.0-upgrade-report` to get the upgrade report.
   `apply boot-2.7-3.0-upgrade-report`

   The file *SPRING_BOOT_3_UPGRADE_REPORT.html* was generated in the project root path.

    ![boot-2.7-3.0-upgrade-report.png](docs/boot-2.7-3.0-upgrade-report.png)

    After applying the recipe, a log of `SBM: applied recipe 'boot-2.7-3.0-upgrade-report'` was committed locally.

### Run and verify with Spring Boot 3

[Re-run and verify](#run-and-verify) again.
