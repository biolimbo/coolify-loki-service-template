# Loki Service Template for Coolify

> ⚠️ **Production Readiness Warning** ⚠️  
> This configuration currently uses in-memory storage for the key-value store (`kvstore: { store: inmemory }`), which may result in data loss during restarts. This setup is not recommended for production use without proper persistent storage configuration.
>
> 🤝 **Open to Collaboration** 🤝  
> This project welcomes contributions! Feel free to submit pull requests, report issues, or suggest improvements.

This repository contains a Docker Compose configuration for running Grafana Loki, a horizontally-scalable, highly-available, multi-tenant log aggregation system inspired by Prometheus.

## Overview

Loki is a log aggregation system designed to store and query logs from all your applications and infrastructure. This template is specifically designed to:

- Integrate with external MinIO instances for object storage
- Work alongside Grafana for log visualization
- Provide basic authentication for security
- Support retention policies for log management

## Architecture

The stack consists of:

### Core Service

- **Loki**: Log aggregation service (port 3100)
  - Configured to use external S3-compatible storage (MinIO)
  - Basic authentication enabled via Traefik middleware
  - TSDB index with 24-hour period rotation

### Storage Configuration

- **Object Storage**: S3-compatible storage (MinIO) for log chunks
- **Local Storage**: Temporary index caching at `/loki/index` and `/loki/index_cache`
- **KV Store**: Currently using in-memory storage (⚠️ not production-ready)

## Prerequisites

- Docker and Docker Compose installed
- Access to a MinIO instance (or S3-compatible storage)
- A Grafana instance for log visualization
- The following environment variables configured in your deployment platform (e.g., Coolify):
  - `S3_BUCKET_ENDPOINT`
  - `S3_BUCKET_NAME`
  - `S3_BUCKET_ACCESS_KEY`
  - `S3_BUCKET_ACCESS_SECRET`
  - `BASIC_AUTH_HASH`
  - `RETENTION_PERIOD`
  - `SERVICE_FQDN_LOKI_3100`

## Quick Start (Coolify Deployment)

1. **Create a new service in Coolify:**

   - In your Coolify dashboard, create a new "Docker Compose" service
   - Copy the contents of `docker-compose.yml` from this repository into the empty Docker Compose configuration

2. **Configure required environment variables in Coolify:**

   - `SERVICE_FQDN_LOKI_3100`: Your Loki service domain (e.g., `loki.yourdomain.com`)
   - `S3_BUCKET_ENDPOINT`: MinIO/S3 endpoint URL (e.g., `s3://minio.yourdomain.com:9000`)
   - `S3_BUCKET_NAME`: Name of the bucket for Loki data
   - `S3_BUCKET_ACCESS_KEY`: Access key for S3/MinIO
   - `S3_BUCKET_ACCESS_SECRET`: Secret key for S3/MinIO
   - `BASIC_AUTH_HASH`: Basic auth hash (see Authentication section below)
   - `RETENTION_PERIOD`: Log retention period (e.g., `744h` for 31 days)

3. **Deploy the stack:**

   - Click "Deploy" in Coolify to start the service
   - Wait for health checks to pass

4. **Configure Grafana:**
   - Add Loki as a data source in Grafana
   - Use the URL: `http://<username>:<password>@loki:3100`
   - Or configure basic auth in Grafana's data source settings

## Configuration

### Authentication

The `BASIC_AUTH_HASH` environment variable is used for basic authentication. Generate it using:

```bash
htpasswd -nbB username password
```

**Important**: The generated hash may contain special characters that need to be properly escaped. You might need to hardcode the value directly in your environment configuration to avoid auto-escaping issues.

Example:

```bash
htpasswd -nbB admin mypassword
# Output: admin:$2y$05$abcdefghijklmnopqrstuvwxyz...
```

### Storage Configuration

Current configuration uses:

- **Object Storage**: External S3/MinIO for log chunks
- **Local Cache**: Temporary index storage
- **KV Store**: In-memory (⚠️ not persistent)

For production use, consider:

- Configuring a persistent KV store (e.g., Consul, etcd)
- Setting up proper backup strategies
- Implementing high availability configurations

### Retention Policy

The `RETENTION_PERIOD` variable controls how long logs are retained. Format examples:

- `24h` - 24 hours
- `7d` - 7 days
- `744h` - 31 days

## Health Checks

The service includes health checks that:

- Test the `/ready` endpoint every 30 seconds
- Allow 15 seconds for initial startup
- Retry up to 5 times before marking as unhealthy

## Volumes

- `loki-data`: Local volume for temporary index and cache storage
- Configuration is embedded in the compose file for easy deployment

## Integration

### With Grafana

1. Add Loki as a data source in Grafana
2. Configure the URL with basic auth credentials
3. Start exploring your logs with LogQL queries

### With Promtail or other log shippers

Configure your log shippers to send logs to:

```
http://username:password@your-loki-domain:3100/loki/api/v1/push
```

## Troubleshooting

### Common Issues

1. **Authentication failures:**

   - Verify `BASIC_AUTH_HASH` is properly escaped
   - Check Traefik middleware configuration
   - Test with curl: `curl -u username:password http://your-loki-domain/ready`

2. **Storage connection issues:**

   - Verify MinIO/S3 credentials
   - Check network connectivity to storage backend
   - Ensure bucket exists and has proper permissions

3. **Data persistence:**
   - Remember that KV store is currently in-memory
   - Index rebuilds may occur after restarts
   - Consider implementing persistent KV store for production

### Logs

**In Coolify:**

- View logs through the Coolify dashboard
- Monitor Loki's startup and query performance
- Check for storage connectivity issues

## Known Limitations

1. **In-Memory KV Store**: Current configuration uses in-memory storage for the ring KV store, which means:

   - State is lost on restart
   - Not suitable for multi-instance deployments
   - May cause temporary query unavailability after restarts

2. **Single Instance**: This template runs a single Loki instance
   - No high availability
   - Limited scalability
   - Consider distributed mode for production

## Future Improvements

- [ ] Add persistent KV store configuration (Consul/etcd)
- [ ] Multi-instance deployment support
- [ ] Automated backup configuration
- [ ] Prometheus metrics exposure
- [ ] Advanced retention policies per tenant

## Contributing

We welcome contributions to improve this template! Areas of focus:

- 🔧 **Production-ready configurations** - Help make this suitable for production use
- 📚 **Documentation improvements** - Better examples and explanations
- 🏗️ **Architecture enhancements** - Multi-instance support, HA configurations
- 🔐 **Security improvements** - Enhanced authentication options
- 📦 **Integration examples** - Promtail, Fluentd, and other log shipper configurations

### Pull Request Guidelines

- Test your changes thoroughly
- Document any new environment variables
- Consider backward compatibility
- Include examples where applicable

## License

This template is provided as-is. Loki itself is licensed under the Apache License 2.0.

## Support

For issues related to:

- **Loki application**: Visit the [official Loki documentation](https://grafana.com/docs/loki/)
- **This Docker setup**: Create an issue in this repository
- **Coolify deployment**: Check the Coolify documentation
