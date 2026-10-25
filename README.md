# LANfinity

**Your LAN, anywhere.**

LANfinity is a lightweight, open-source reverse tunneling tool written in Rust. It lets other people connect to games and services running on your computer without requiring port forwarding, complicated router configuration, or a VPN.

The original idea came from a pretty simple problem: sharing a Minecraft singleplayer world with friends shouldn't require setting up a dedicated server, installing a bunch of mods, or figuring out how your router works.

LANfinity aims to make that process as straightforward as possible while remaining useful for other applications that need incoming network connections.

**LANfinity is currently in early development.** The networking prototype is being built first, with Minecraft integration and a proper desktop application planned for later versions.

## How it works

LANfinity uses a reverse tunnel between your computer and an edge relay.

Instead of requiring someone on the Internet to connect directly to your home network, the LANfinity agent establishes an outbound connection to a relay server. Other clients connect to that relay, which forwards their traffic through the existing tunnel to the service running on your computer.

```text
Your Computer                        LANfinity Edge

Minecraft / Local Service
          |
          |
    LANfinity Agent
          |
          | Outbound Tunnel
          |
          +----------------------> Edge Relay
                                       |
                                       |
                                  Remote Client
```

Your computer remains responsible for running the actual application. The relay's job is simply to move network traffic between the two endpoints.

This approach avoids the need to configure inbound port forwarding on your router.

For applications such as Minecraft Java Edition, joining players would only need the game's normal multiplayer connection feature. They wouldn't need to install LANfinity themselves.

## Why LANfinity?

There are already plenty of ways to connect computers over the Internet, but many of them involve more setup than necessary for something as simple as inviting a friend into a game.

LANfinity is being developed around a few principles:

- **No port forwarding.** The host establishes an outbound connection, so there's no need to open ports on a home router.
- **No client-side installation.** For supported TCP applications, remote users connect through the relay using their existing client.
- **Lightweight.** The agent and relay are written in Rust, without requiring Java, Node.js, or Python runtimes.
- **Cross-platform.** Windows, Linux, and macOS are the intended platforms.
- **Application-independent networking.** The tunneling layer isn't tied to Minecraft and can eventually support other TCP applications.
- **Open source.** The goal is to keep the networking implementation understandable, auditable, and maintainable.

LANfinity isn't trying to replace every VPN or networking tool. It's focused on making individual locally hosted services accessible with as little friction as possible.

## Examples

### Minecraft Java Edition

Suppose you're playing Minecraft in a singleplayer world and want a friend to join.

Normally, opening a world to LAN only allows other devices on your local network to connect. Unless you set up additional networking software or port forwarding, somebody outside your home can't simply join it.

With the planned LANfinity integration, the process would look like this:

1. Open your Minecraft world.
2. Select **Open to LAN**.
3. LANfinity detects the local game server.
4. Start sharing the world through LANfinity.
5. Send your friend the assigned server address.
6. Your friend joins through Minecraft's normal multiplayer menu.

For example, a future LANfinity address might look like:

```text
survival.mc.example.net
```

That address is illustrative, not an active LANfinity endpoint.

Your friend connects to the relay, and LANfinity carries the connection back to your Minecraft world.

The world is still hosted on your computer. LANfinity does not run Minecraft for you or replace the game's integrated server.

The same concept should work with compatible modded Minecraft installations, although everyone still needs the appropriate game version and required mods.

### Other game servers

Minecraft is the starting point, but the underlying system is intended to work with other TCP-based game servers.

For example, if a game server is running locally on:

```text
127.0.0.1:27015
```

LANfinity could be configured to forward connections from an assigned public endpoint to that local service.

The application would continue running normally on the host machine while LANfinity handles the connection between external players and the local server.

Not every game will work automatically. Some games use UDP, require additional ports, or depend on discovery and authentication systems that need separate support.

### Local development services

LANfinity could also be useful for temporarily exposing development services.

For example, you might have a local application listening on:

```text
127.0.0.1:8080
```

Instead of changing router settings, you could create a temporary tunnel and make it accessible to another device through a relay.

This could be useful for testing multiplayer backends, sharing development builds, or working with a service running on another network.

Publicly accessible services will require appropriate authentication, transport security, and access restrictions.

## Architecture

LANfinity is being developed as several small Rust components.

| Component | Responsibility |
|---|---|
| `lanfinity-agent` | Runs on the host computer, establishes outbound connections, and forwards traffic to local services |
| `lanfinity-relay` | Accepts agent and remote-client connections and routes traffic between them |
| `lanfinity-protocol` | Defines the shared networking protocol used by the agent and relay |
| Control plane (planned) | Manages tunnel registration, addressing, relay selection, and configuration |
| Desktop application (planned) | Provides an interface for sharing services and managing connections |

The relay is designed to be as application-independent as possible. It shouldn't need to understand Minecraft packets, HTTP requests, or the internal protocol of every application using the tunnel.

The long-term goal is to separate the management system from the actual data path. Once a connection has been established, forwarding traffic shouldn't require a database lookup or an API request for every packet.

## Development status

LANfinity is still experimental. The current Rust workspace contains the initial networking components, and development is focused on getting reliable agent-to-relay communication working before introducing public infrastructure.

The planned development order is:

| Phase | Goal |
|---|---|
| 0 | Audit and organize the existing Rust workspace |
| 1 | Build a working reverse TCP tunnel |
| 2 | Support multiple connections and reliable session handling |
| 3 | Introduce authenticated, encrypted transport |
| 4 | Add Minecraft Open-to-LAN integration |
| 5 | Implement tunnel registration and public addressing |
| 6 | Test Windows, Linux, and macOS |
| 7 | Build a lightweight desktop interface |
| 8 | Run a private beta |
| 9 | Benchmark and optimize the networking implementation |
| 10 | Prepare public relay infrastructure |

These are development goals, not a list of features already available.

In particular, the current experimental builds should not be treated as secure public tunneling software. Authentication, encryption, and operational safeguards need to be implemented and tested before Internet-facing deployment.

## Building from source

LANfinity uses Rust and Cargo.

Once Rust is installed, clone the repository and enter the project directory.

From the workspace root:

```bash
cargo check --workspace
cargo build --workspace
cargo test --workspace
```

During development, the agent and relay can be run separately:

```bash
cargo run -p portless-relay
```

```bash
cargo run -p portless-agent
```

These are the existing prototype package names. They will be renamed as the LANfinity migration progresses.

The current binaries are development components, not a finished application. Running them does not automatically create a public tunnel or assign a hostname.

## Performance

LANfinity is intended to have a relatively small resource footprint, especially on the host computer.

The project uses Rust and asynchronous networking to minimize unnecessary overhead while supporting concurrent connections.

Some of the main design priorities are:

- Avoiding unnecessary packet inspection.
- Keeping memory allocations and buffering under control.
- Handling concurrent connections efficiently.
- Avoiding unnecessary compression and data transformations.
- Keeping the relay's forwarding path independent of the control plane.
- Measuring latency, throughput, and resource usage before optimizing.

Actual performance benchmarks will be published once the networking implementation is sufficiently complete.

## Future plans

There are several features worth exploring after the basic tunneling system becomes reliable.

These include automatic Minecraft version and mod-loader detection, whitelists, temporary sharing links, relay-region selection, custom domains, connection diagnostics, and support for additional games.

Direct peer-to-peer connections with relay fallback may also be worth exploring later.

The immediate priority is making the core tunnel secure, stable, and easy to use. Additional features shouldn't come at the expense of that.

## Contributing

LANfinity is an early-stage project, so bug reports, testing, and feedback are especially useful.

Areas where contributions may eventually be helpful include:

- Rust networking and protocol development
- Windows, Linux, and macOS compatibility
- Minecraft integration
- Integration testing
- Security reviews
- Performance benchmarking
- Documentation and usability

Before contributing large changes, please open an issue or discussion to explain the proposed approach.

## Project philosophy

LANfinity started with the idea that connecting to someone's locally hosted game should be simple.

The networking behind that can be complicated, but the person using it shouldn't have to deal with most of that complexity.

The goal isn't to build the largest networking platform or cram every possible feature into one application. It's to create a small, reliable tool that does its job well.

**Your LAN, anywhere.**
