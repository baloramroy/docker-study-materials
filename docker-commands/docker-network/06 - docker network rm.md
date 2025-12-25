# `docker network rm`

## Description
**Removes** one or more **networks** by **name or identifier**. To remove a network, you must first **disconnect** any containers connected to it.

**Usage**

```
docker network rm NETWORK [NETWORK...]
```

**Aliases**

```bash
docker network remove
```
## Examples

**To remove the network named 'my-network':**

```bash
docker network rm my-network
```
**The following example deletes a network with id 3695c422697f and a network named my-network:**

```bash
docker network rm 3695c422697f my-network
```
