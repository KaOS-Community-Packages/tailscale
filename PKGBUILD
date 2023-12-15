pkgname=tailscale
pkgver=1.56.0
pkgrel=1
pkgdesc="A mesh VPN that makes it easy to connect your devices, wherever they are."
arch=("x86_64")
url="https://pkgs.tailscale.com/"
license=("MIT")
makedepends=( "go")
depends=("glibc")
backup=("etc/default/tailscaled")
source=("${pkgname}-${pkgver}.tar.gz::https://github.com/tailscale/${pkgname}/archive/refs/tags/v${pkgver}.tar.gz") 
sha256sums=('SKIP')

prepare() {
    cd "${pkgname}-${pkgver}/"
    go mod vendor
}

build() {
    cd "${pkgname}-${pkgver}/"
    export CGO_CPPFLAGS="${CPPFLAGS}"
    export CGO_CFLAGS="${CFLAGS}"
    export CGO_CXXFLAGS="${CXXFLAGS}"
    export CGO_LDFLAGS="${LDFLAGS}"
    # pacman bug
    # export GOPATH="${srcdir}/"
    export GOFLAGS="-buildmode=pie -mod=readonly -modcacherw"
    GO_LDFLAGS="\
        -compressdwarf=false \
        -linkmode=external \
        -X tailscale.com/version.Long=${pkgver} \
        -X tailscale.com/version.Short=$(cut -d+ -f1 <<< "${pkgver}")"
    for cmd in ./cmd/tailscale ./cmd/tailscaled; do
        go build -v -tags xversion -ldflags "$GO_LDFLAGS" "$cmd"
    done
}

#TODO: Figure out why tests are failing
# check() {
#     cd "${pkgname}/"
#     go test $(go list ./... | grep -v tsdns_test)
# }

package() {
    cd "${srcdir}/${pkgname}-${pkgver}/"
    install -dm755 "${pkgdir}/usr/bin/"
    install -Dm755 ./tailscale,{tailscaled} -t "${pkgdir}/usr/bin/"
    install -Dm644 ./cmd/tailscaled/tailscaled.defaults "${pkgdir}/etc/default/tailscaled"
    install -Dm644 ./cmd/tailscaled/tailscaled.service -t "${pkgdir}/usr/lib/systemd/system/"
    install -Dm644 ./LICENSE -t "${pkgdir}/usr/share/licenses/${pkgname}/"
    install -dm755 "${pkgdir}/sbin"
    install -dm755 "${pkgdir}/usr/sbin"
    ln -f "${pkgdir}/usr/bin/tailscaled" "--target-directory=${pkgdir}/sbin/"
    ln -f "${pkgdir}/usr/bin/tailscaled" "--target-directory=${pkgdir}/usr/sbin/"
}
