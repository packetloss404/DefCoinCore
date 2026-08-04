Release Process
====================

> **Note:** This document was inherited from the upstream Litecoin Core
> release process and adapted for Defcoin Core. The general workflow is
> the same; project names, repository URLs, binary names, and the final
> publication endpoints have been retargeted at Defcoin Core. Sections
> that still describe upstream team conventions (translations, snap
> packaging on Launchpad, etc.) are kept here as a starting point — adjust
> to match the Defcoin Core maintainer setup before relying on them.

## Branch updates

### Before every release candidate

* Update translations (the [translation process](translation_process.md) describes the synchronisation flow; reach out to a Defcoin Core maintainer if you need an owner for this).
* Update manpages, see [gen-manpages.sh](../contrib/devtools/gen-manpages.sh).
* Update release candidate version in `configure.ac` (`CLIENT_VERSION_RC`).

### Before every major and minor release

* Update [bips.md](bips.md) to account for changes since the last release (don't forget to bump the version number on the first line).
* Update version in `configure.ac` (don't forget to set `CLIENT_VERSION_RC` to `0`).
* Write release notes (see "Write the release notes" below).

### Before every major release

* On both the master branch and the new release branch:
  - update `CLIENT_VERSION_MINOR` in [`configure.ac`](../configure.ac)
  - update `CLIENT_VERSION_MINOR`, `PACKAGE_VERSION`, and `PACKAGE_STRING` in `build_msvc/defcoin_config.h` (the upstream-style file is currently still tracked under its old name; see `recommendations.md` for the rename plan)
* On the new release branch in [`configure.ac`](../configure.ac) and `build_msvc/defcoin_config.h`:
  - set `CLIENT_VERSION_REVISION` to `0`
  - set `CLIENT_VERSION_IS_RELEASE` to `true`

#### Before branch-off

* Update hardcoded [seeds](../contrib/seeds/README.md).
* Update [`src/chainparams.cpp`](../src/chainparams.cpp) `m_assumed_blockchain_size` and `m_assumed_chain_state_size` with the current size plus some overhead (see [this](#how-to-calculate-assumed-blockchain-and-chain-state-size) for information on how to calculate them).
* Update [`src/chainparams.cpp`](../src/chainparams.cpp) `chainTxData` with statistics about the transaction count and rate. Use the output of the `getchaintxstats` RPC.
* Update `src/chainparams.cpp` `nMinimumChainWork` and `defaultAssumeValid` (and the block height comment) with information from the `getblockheader` (and `getblockhash`) RPCs.
  - The selected value must not be orphaned so it may be useful to set the value two blocks back from the tip.
  - Testnet should be set some tens of thousands back from the tip due to reorgs there.
  - This update should be reviewed with a reindex-chainstate with assumevalid=0 to catch any defect
     that causes rejection of blocks in the past history.
- Clear the release notes and move them to the wiki (see "Write the release notes" below).

#### After branch-off (on master)

- Update the version of `contrib/gitian-descriptors/*.yml`.

#### After branch-off (on the major release branch)

- Update the versions.
- Create a pinned meta-issue for testing the release candidate and provide a link to it in the release announcements where useful.

#### Before final release

- Merge the release notes from the wiki into the branch.
- Ensure the "Needs release note" label is removed from all relevant pull requests and issues.


## Building

### First time / New builders

If you're using the automated script (found in `contrib/gitian-build.py` if it has been added to this tree; see `doc/gitian-building.md` for upstream guidance), then at this point you should run it with the "--setup" command. Otherwise ignore this.

Check out the source code in the following directory hierarchy.

    cd /path/to/your/toplevel/build
    git clone https://github.com/devrandom/gitian-builder.git
    git clone https://github.com/packetloss404/DefCoinCore.git

Signatures and detached-signature repos for Defcoin Core will be added
once a Gitian-based release pipeline is set up. Until then, signed
binaries are produced via a manual process — see the project owner for
the current procedure.

### Defcoin Core maintainers/release engineers, suggestion for writing release notes

Write the release notes. `git shortlog` helps a lot, for example:

    git shortlog --no-merges v(current version, e.g. 26.6.7)..v(new version, e.g. 26.6.8)

Generate list of authors:

    git log --format='- %aN' v(current version, e.g. 26.6.7)..v(new version, e.g. 26.6.8) | sort -fiu

Tag the version (or release candidate) in git:

    git tag -s v(new version, e.g. 26.6.8)

### Setup and perform Gitian builds

If you're using the automated script (see the "First time / New builders" note above), then at this point you should run it with the "--build" command. Otherwise ignore this.

Setup Gitian descriptors:

    pushd ./DefCoinCore
    export SIGNER="(your Gitian key, ie bluematt, sipa, etc)"
    export VERSION=(new version, e.g. 26.6.8)
    git fetch
    git checkout v${VERSION}
    popd

### Fetch and create inputs: (first time, or when dependency versions change)

    pushd ./gitian-builder
    mkdir -p inputs
    wget -O inputs/osslsigncode-2.0.tar.gz https://github.com/mtrojnar/osslsigncode/archive/2.0.tar.gz
    echo '5a60e0a4b3e0b4d655317b2f12a810211c50242138322b16e7e01c6fbb89d92f inputs/osslsigncode-2.0.tar.gz' | sha256sum -c
    popd

Create the macOS SDK tarball, see the [macdeploy instructions](../contrib/macdeploy/README.md#deterministic-macos-dmg-notes) for details, and copy it into the inputs directory.

### Optional: Seed the Gitian sources cache and offline git repositories

NOTE: Gitian is sometimes unable to download files. If you have errors, try the step below.

By default, Gitian will fetch source files as needed. To cache them ahead of time, make sure you have checked out the tag you want to build in DefCoinCore, then:

    pushd ./gitian-builder
    make -C ../DefCoinCore/depends download SOURCES_PATH=`pwd`/cache/common
    popd

Only missing files will be fetched, so this is safe to re-run for each build.

NOTE: Offline builds must use the --url flag to ensure Gitian fetches only from local URLs. For example:

    pushd ./gitian-builder
    ./bin/gbuild --url DefCoinCore=/path/to/DefCoinCore,signature=/path/to/sigs {rest of arguments}
    popd

The gbuild invocations below <b>DO NOT DO THIS</b> by default.

### Build and sign Defcoin Core for Linux, Windows, and macOS:

    export GITIAN_THREADS=2
    export GITIAN_MEMORY=3000

    pushd ./gitian-builder
    ./bin/gbuild --num-make $GITIAN_THREADS --memory $GITIAN_MEMORY --commit DefCoinCore=v${VERSION} ../DefCoinCore/contrib/gitian-descriptors/gitian-linux.yml
    ./bin/gsign --signer "$SIGNER" --release ${VERSION}-linux --destination ../gitian.sigs.dfc/ ../DefCoinCore/contrib/gitian-descriptors/gitian-linux.yml
    mv build/out/defcoin-*.tar.gz build/out/src/defcoin-*.tar.gz ../

    ./bin/gbuild --num-make $GITIAN_THREADS --memory $GITIAN_MEMORY --commit DefCoinCore=v${VERSION} ../DefCoinCore/contrib/gitian-descriptors/gitian-win.yml
    ./bin/gsign --signer "$SIGNER" --release ${VERSION}-win-unsigned --destination ../gitian.sigs.dfc/ ../DefCoinCore/contrib/gitian-descriptors/gitian-win.yml
    mv build/out/defcoin-*-win-unsigned.tar.gz inputs/defcoin-win-unsigned.tar.gz
    mv build/out/defcoin-*.zip build/out/defcoin-*.exe ../

    ./bin/gbuild --num-make $GITIAN_THREADS --memory $GITIAN_MEMORY --commit DefCoinCore=v${VERSION} ../DefCoinCore/contrib/gitian-descriptors/gitian-osx.yml
    ./bin/gsign --signer "$SIGNER" --release ${VERSION}-osx-unsigned --destination ../gitian.sigs.dfc/ ../DefCoinCore/contrib/gitian-descriptors/gitian-osx.yml
    mv build/out/defcoin-*-osx-unsigned.tar.gz inputs/defcoin-osx-unsigned.tar.gz
    mv build/out/defcoin-*.tar.gz build/out/defcoin-*.dmg ../
    popd

Build output expected:

  1. source tarball (`defcoin-${VERSION}.tar.gz`)
  2. linux 32-bit and 64-bit dist tarballs (`defcoin-${VERSION}-linux[32|64].tar.gz`)
  3. windows 32-bit and 64-bit unsigned installers and dist zips (`defcoin-${VERSION}-win[32|64]-setup-unsigned.exe`, `defcoin-${VERSION}-win[32|64].zip`)
  4. macOS unsigned installer and dist tarball (`defcoin-${VERSION}-osx-unsigned.dmg`, `defcoin-${VERSION}-osx64.tar.gz`)
  5. Gitian signatures (in `gitian.sigs.dfc/${VERSION}-<linux|{win,osx}-unsigned>/(your Gitian key)/`)

### Verify other gitian builders signatures to your own. (Optional)

Add other gitian builders keys to your gpg keyring, and/or refresh keys: See `../DefCoinCore/contrib/gitian-keys/README.md`.

Verify the signatures

    pushd ./gitian-builder
    ./bin/gverify -v -d ../gitian.sigs.dfc/ -r ${VERSION}-linux ../DefCoinCore/contrib/gitian-descriptors/gitian-linux.yml
    ./bin/gverify -v -d ../gitian.sigs.dfc/ -r ${VERSION}-win-unsigned ../DefCoinCore/contrib/gitian-descriptors/gitian-win.yml
    ./bin/gverify -v -d ../gitian.sigs.dfc/ -r ${VERSION}-osx-unsigned ../DefCoinCore/contrib/gitian-descriptors/gitian-osx.yml
    popd

### Next steps:

Commit your signature to gitian.sigs.dfc:

    pushd gitian.sigs.dfc
    git add ${VERSION}-linux/"${SIGNER}"
    git add ${VERSION}-win-unsigned/"${SIGNER}"
    git add ${VERSION}-osx-unsigned/"${SIGNER}"
    git commit -m "Add ${VERSION} unsigned sigs for ${SIGNER}"
    git push  # Assuming you can push to the gitian.sigs tree
    popd

Codesigner only: Create Windows/macOS detached signatures:
- Only one person handles codesigning. Everyone else should skip to the next step.
- Only once the Windows/macOS builds each have 3 matching signatures may they be signed with their respective release keys.

Codesigner only: Sign the macOS binary:

    transfer defcoin-osx-unsigned.tar.gz to macOS for signing
    tar xf defcoin-osx-unsigned.tar.gz
    ./detached-sig-create.sh -s "Key ID"
    Enter the keychain password and authorize the signature

    Now a manual deterministic disk image (dmg) creation is required.

    First time setup for codesigner, requires creation of app-specific-password via Apple ID website.
    Once password is obtained, save it to the macOS Keychain for future reference:

    $   xcrun altool -u "<apple-id-email>" -p "<app-specific-password>" --store-password-in-keychain-item "<apple-id-notarisation-app-specific-password>"

    If <team-id-shortcode> is unknown for team accounts with multiple organisations, query:

    $   xcrun altool --list-providers -u "<apple-id-email>" -p "@keychain:<apple-id-notarisation-app-specific-password>"

    Notarize the disk image:

    $   xcrun altool --notarize-app --primary-bundle-id "org.defcoin.Defcoin-Qt" -u "<apple-id-email>" -p "@keychain:<apple-id-notarisation-app-specific-password>" --asc-provider <team-id-shortcode> -t osx -f defcoin-${VERSION}-osx.dmg

    The notarization takes a few minutes. Check the status:

    $   xcrun altool --notarization-info <request-uuid> -u "<apple-id-email>" -p "@keychain:<apple-id-notarisation-app-specific-password>" --asc-provider <team-id-shortcode>

    If notarization fails, query log with uuid:

    $   xcrun altool --notarization-info <request-uuid> -u "<apple-id-email>" -p "@keychain:<apple-id-notarisation-app-specific-password>" --asc-provider <team-id-shortcode>

    Staple the notarization ticket onto the application

    $   xcrun stapler staple dist/Defcoin-Qt.app

Codesigner only: Sign the windows binaries:

    tar xf defcoin-win-unsigned.tar.gz
    ./detached-sig-create.sh -key /path/to/codesign.key
    Enter the passphrase for the key when prompted
    signature-win.tar.gz will be created

Codesigner only: Commit the detached codesign payloads:

    cd ~/defcoin-detached-sigs
    #checkout the appropriate branch for this release series
    rm -rf *
    tar xf signature-osx.tar.gz
    tar xf signature-win.tar.gz
    #copy the notarization ticket to detached-sigs repo
    cp dist/Defcoin-Qt.app/Contents/CodeResources osx/dist/Defcoin-Qt.app/Contents/
    git add -A
    git commit -m "point to ${VERSION}"
    git tag -s v${VERSION} HEAD
    git push the current branch and new tag

Non-codesigners: wait for Windows/macOS detached signatures:

- Once the Windows/macOS builds each have 3 matching signatures, they will be signed with their respective release keys.
- Detached signatures will then be committed to the defcoin-detached-sigs repository, which can be combined with the unsigned apps to create signed binaries.

Create (and optionally verify) the signed macOS binary:

    pushd ./gitian-builder
    ./bin/gbuild -i --commit signature=v${VERSION} ../DefCoinCore/contrib/gitian-descriptors/gitian-osx-signer.yml
    ./bin/gsign --signer "$SIGNER" --release ${VERSION}-osx-signed --destination ../gitian.sigs.dfc/ ../DefCoinCore/contrib/gitian-descriptors/gitian-osx-signer.yml
    ./bin/gverify -v -d ../gitian.sigs.dfc/ -r ${VERSION}-osx-signed ../DefCoinCore/contrib/gitian-descriptors/gitian-osx-signer.yml
    mv build/out/defcoin-osx-signed.dmg ../defcoin-${VERSION}-osx.dmg
    popd

Create (and optionally verify) the signed Windows binaries:

    pushd ./gitian-builder
    ./bin/gbuild -i --commit signature=v${VERSION} ../DefCoinCore/contrib/gitian-descriptors/gitian-win-signer.yml
    ./bin/gsign --signer "$SIGNER" --release ${VERSION}-win-signed --destination ../gitian.sigs.dfc/ ../DefCoinCore/contrib/gitian-descriptors/gitian-win-signer.yml
    ./bin/gverify -v -d ../gitian.sigs.dfc/ -r ${VERSION}-win-signed ../DefCoinCore/contrib/gitian-descriptors/gitian-win-signer.yml
    mv build/out/defcoin-*win64-setup.exe ../defcoin-${VERSION}-win64-setup.exe
    popd

Commit your signature for the signed macOS/Windows binaries:

    pushd gitian.sigs.dfc
    git add ${VERSION}-osx-signed/"${SIGNER}"
    git add ${VERSION}-win-signed/"${SIGNER}"
    git commit -m "Add ${SIGNER} ${VERSION} signed binaries signatures"
    git push  # Assuming you can push to the gitian.sigs.dfc tree
    popd

### After 3 or more people have gitian-built and their results match:

- Create `SHA256SUMS.asc` for the builds, and GPG-sign it:

```bash
sha256sum * > SHA256SUMS
```

The list of files should be:
```
defcoin-${VERSION}-aarch64-linux-gnu.tar.gz
defcoin-${VERSION}-arm-linux-gnueabihf.tar.gz
defcoin-${VERSION}-riscv64-linux-gnu.tar.gz
defcoin-${VERSION}-x86_64-linux-gnu.tar.gz
defcoin-${VERSION}-osx64.tar.gz
defcoin-${VERSION}-osx.dmg
defcoin-${VERSION}.tar.gz
defcoin-${VERSION}-win64-setup.exe
defcoin-${VERSION}-win64.zip
```
The `*-debug*` files generated by the gitian build contain debug symbols
for troubleshooting by developers. It is assumed that anyone that is interested
in debugging can run gitian to generate the files for themselves. To avoid
end-user confusion about which file to pick, as well as save storage
space *do not upload these to the defcoin distribution server, nor put them in the torrent*.

- GPG-sign it, delete the unsigned file:
```
gpg --digest-algo sha256 --clearsign SHA256SUMS # outputs SHA256SUMS.asc
rm SHA256SUMS
```
(the digest algorithm is forced to sha256 to avoid confusion of the `Hash:` header that GPG adds with the SHA256 used for the files)
Note: check that SHA256SUMS itself doesn't end up in SHA256SUMS, which is a spurious/nonsensical entry.

- Upload zips and installers, as well as `SHA256SUMS.asc` from last step, to the defcoin distribution server.

- Update the defcoin distribution version metadata.

- Update other repositories and websites for new version

  - Update title of the project IRC channels on Libera.Chat
    (#defcoin, #defcoin-dev) — coordinate with the channel owners.

  - Update any package-management metadata for the project (e.g.
    Flatpak, Snap, AUR).

  - Archive release notes for the new version to `doc/release-notes/`
    (branch `master` and branch of the release)

  - Create a [new GitHub release](https://github.com/packetloss404/DefCoinCore/releases/new) with a link to the archived release notes.

  - Celebrate

### Additional information

#### <a name="how-to-calculate-assumed-blockchain-and-chain-state-size"></a>How to calculate `m_assumed_blockchain_size` and `m_assumed_chain_state_size`

Both variables are used as a guideline for how much space the user needs on their drive in total, not just strictly for the blockchain.
Note that all values should be taken from a **fully synced** node and have an overhead of 5-10% added on top of its base value.

To calculate `m_assumed_blockchain_size`:
- For `mainnet` -> Take the size of the data directory, excluding `/regtest` and `/testnet4` directories.
- For `testnet` -> Take the size of the `/testnet4` directory.


To calculate `m_assumed_chain_state_size`:
- For `mainnet` -> Take the size of the `/chainstate` directory.
- For `testnet` -> Take the size of the `/testnet4/chainstate` directory.

Notes:
- When taking the size for `m_assumed_blockchain_size`, there's no need to exclude the `/chainstate` directory since it's a guideline value and an overhead will be added anyway.
- The expected overhead for growth may change over time, so it may not be the same value as last release; pay attention to that when changing the variables.
