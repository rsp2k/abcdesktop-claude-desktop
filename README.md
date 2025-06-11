# Claude Desktop for abcdesktop.io

This project creates an abcdesktop application container for Claude Desktop, allowing you to run Claude Desktop in a web browser through the abcdesktop platform.

## What is abcdesktop.io?

abcdesktop.io is a cloud-native desktop service that runs applications in containers and provides access through a web browser. It's built on Kubernetes and allows secure, isolated execution of desktop applications.

## What is Claude Desktop?

Claude Desktop is an unofficial Linux port of Anthropic's Claude AI assistant desktop application. It's an Electron-based app that provides a native desktop experience for interacting with Claude.

## Quick Start

1. **Clone this repository or create the build script:**
   ```bash
   wget https://raw.githubusercontent.com/yourusername/claude-desktop-abcdesktop/main/build.sh
   chmod +x build.sh
   ./build.sh
   ```

2. **Build the Docker image:**
   ```bash
   cd claude-desktop-abcdesktop
   docker build -t claude-desktop-abcdesktop .
   ```

3. **Test locally (optional):**
   ```bash
   docker run -it --rm \
     -e DISPLAY=$DISPLAY \
     -v /tmp/.X11-unix:/tmp/.X11-unix \
     claude-desktop-abcdesktop \
     /opt/Claude/claude
   ```

4. **Push to registry:**
   ```bash
   docker tag claude-desktop-abcdesktop:latest yourdockerhub/claude-desktop-abcdesktop:latest
   docker push yourdockerhub/claude-desktop-abcdesktop:latest
   ```

5. **Deploy to abcdesktop:**
   ```bash
   # Update the image path in claude-desktop-app.json
   sed -i 's|yourdockerhub|your-actual-dockerhub-username|g' claude-desktop-app.json
   
   # Deploy
   curl -X PUT -H 'Content-Type: application/json' \
     http://your-abcdesktop-host:30443/API/manager/image \
     -d @claude-desktop-app.json
   ```

## Project Structure

```
claude-desktop-abcdesktop/
├── Dockerfile              # Main Dockerfile for building the image
├── icons/
│   └── claude-desktop.svg  # Application icon
├── claude-desktop-app.json # abcdesktop deployment configuration
└── README.md              # This file
```

## Configuration Details

### Docker Labels

The Dockerfile includes specific labels required by abcdesktop:

- `oc.icon`: Icon filename
- `oc.icondata`: Base64-encoded icon data
- `oc.keyword`: Search keywords
- `oc.cat`: Application category
- `oc.launch`: X11 window class
- `oc.path`: Executable path
- `oc.displayname`: Display name in abcdesktop
- `oc.host_config`: Container resource limits

### Resource Requirements

- **Memory**: 4GB recommended (configured in host_config)
- **Shared Memory**: 2GB (for Electron/Chromium)
- **Storage**: ~3GB for the image

## Troubleshooting

### Build Failures

1. **Node.js version issues:**
   - The build requires Node.js 20.x
   - If you get npm errors, try using Node.js 18.x instead

2. **Network timeouts:**
   - The build downloads several large files
   - Use a reliable internet connection
   - Consider using a build server closer to package repositories

3. **Missing dependencies:**
   - The Dockerfile includes most common dependencies
   - Check the Claude Desktop GitHub for updated requirements

### Runtime Issues

1. **Application won't start:**
   ```bash
   # Check if the executable exists
   docker run --rm claude-desktop-abcdesktop ls -la /opt/Claude/
   ```

2. **X11 window class mismatch:**
   - Run the app and use `wmctrl -lx` to get the correct class
   - Update `oc.launch` label in Dockerfile

3. **Authentication issues:**
   - Claude Desktop requires Anthropic account authentication
   - Users need to sign in on first launch
   - Credentials are stored in the container (not persistent by default)

### abcdesktop Integration

1. **Application not appearing:**
   - Check abcdesktop logs: `kubectl logs -n abcdesktop deployment/pyos`
   - Verify image was pulled: `kubectl get pods -n abcdesktop`

2. **Icon not showing:**
   - Ensure icon is properly base64 encoded
   - Check icon file is copied to correct location

## Advanced Configuration

### Persistent Storage

To maintain Claude Desktop settings and authentication:

```yaml
# Add to abcdesktop user pod configuration
volumes:
  - name: claude-config
    persistentVolumeClaim:
      claimName: claude-config-pvc
volumeMounts:
  - name: claude-config
    mountPath: /home/balloon/.config/Claude
```

### GPU Acceleration

For better performance with Electron:

```json
{
  "host_config": {
    "devices": ["/dev/dri:/dev/dri"],
    "group_add": ["video"]
  }
}
```

### Custom Builds

To use a specific Claude Desktop version:

1. Fork the claude-desktop repository
2. Checkout desired version
3. Update Dockerfile to use your fork:
   ```dockerfile
   RUN git clone https://github.com/yourusername/claude-desktop.git . && \
       git checkout v1.2.3 && \
       ./install-claude-desktop.sh
   ```

## Security Considerations

- Claude Desktop runs as the `balloon` user (non-root)
- Network access is required for API communication
- Consider network policies if running in production
- Authentication tokens are stored in the container

## Updates

When Claude Desktop updates:

1. Rebuild the Docker image
2. Push new version with updated tag
3. Update abcdesktop application configuration
4. Rolling update will apply to new sessions

## Contributing

Improvements welcome! Please submit issues and pull requests for:

- Better icon/branding
- Build optimizations
- Additional desktop integration features
- Documentation improvements

## License

- Build scripts: MIT License
- Claude Desktop: Check [Anthropic's Terms](https://www.anthropic.com/legal/consumer-terms)
- abcdesktop.io: Apache 2.0 License

## Resources

- [abcdesktop.io Documentation](https://www.abcdesktop.io/)
- [Claude Desktop Linux GitHub](https://github.com/emsi/claude-desktop)
- [abcdesktop Applications](https://github.com/abcdesktopio/oc.apps)
- [Anthropic Claude](https://www.anthropic.com/)
