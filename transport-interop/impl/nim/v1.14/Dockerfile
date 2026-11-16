FROM nimlang/nim:2.2.4-ubuntu-regular AS builder

WORKDIR /app

COPY nim-libp2p-* nim-libp2p

RUN --mount=type=cache,target=/var/cache/apt apt-get update && apt-get install -y libssl-dev

# Install the pinned commits with the legacy solver and the Nim of this image.
# nimble 0.18 resolves them with its SAT solver and may fetch and build a newer
# Nim (2.4.2), which fails on some runs ("Unsatisfiable dependencies", "system
# module needs: raiseOverflow"). install_pinned runs its own nimble install, so
# the same flags are added there.
RUN \
  cd nim-libp2p && \
  sed -i 's/exec "nimble install -y "/exec "nimble install -y --solver:legacy --useSystemNim "/' libp2p.nimble && \
  grep -q 'nimble install -y --solver:legacy --useSystemNim ' libp2p.nimble && \
  nimble --solver:legacy --useSystemNim install_pinned

RUN \
  cd nim-libp2p && \
  nimble --solver:legacy --useSystemNim install -y redis@0.5.0

COPY main.nim main.nim
RUN \
  nim c --NimblePath:nim-libp2p/nimbledeps/pkgs2 -p:nim-libp2p -d:libp2p_quic_support -d:chronicles_log_level=WARN --threads:off main.nim

ENTRYPOINT ["/app/main"]
