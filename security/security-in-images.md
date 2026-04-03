## Security in docker images

### Least Privilege Principle (Princípio do mínimo privilégio)

- ❌ Allow everything and private what is needed
- ✅ Private everyhting and only allow what is needed

### Use cgroups to control container resources

- Limit CPU usage
- Limit RAM usage
- Configure container CPU priority
- Configure disk limits (block I/O - priority)
