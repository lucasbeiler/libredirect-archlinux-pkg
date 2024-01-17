# Maintainer: Lucas S. Beiler <lucasbeiler@protonmail.com>
pkgname=chromium-extension-libredirect
pkgver=2.8.0
pkgrel=1
pkgdesc="Libredirect extension for Chromium"
arch=('any')
url="https://github.com/libredirect/browser_extension"
license=('gpl-3.0')
provides=('chromium-extension-libredirect')
makedepends=(
	'chromium'
	'jq'
	'npm'
	'openssl'
	'sed'
	'web-ext'
)

source=(
	"${pkgname}-${pkgver}.zip::${url}/archive/refs/tags/v${pkgver}.zip"
	"template.json"
)

sha256sums=(
	'4072dd8ebc1a9459851f09e97b8fdda063a9e293b3c037f5385495b2d975ee1b'
	'd3f71f6947174155f9ed31948227009f9a8a7e8c415cf1e56200a4399ad51c67'
)

build() {
	# Generate keys and the extension JSON file from the template.json.
	openssl genrsa 2048 | openssl pkcs8 -topk8 -nocrypt -traditional > private_key.pem
	pub_key=$(openssl rsa -in private_key.pem -pubout -outform DER | base64 -w 0)
	export ext_identifier="$(echo ${pub_key} | base64 -d | sha256sum | head -c32 | tr '0-9a-f' 'a-p')"
	cp template.json ${ext_identifier}.json
	sed -i "s/PKGNAME/${pkgname}/g" ${ext_identifier}.json
	sed -i "s/PKGVER/${pkgver}/g" ${ext_identifier}.json

	# Compile libredirect artifacts and generate the manifest file.
	mkdir libredirect.chromium
	cd browser_extension-${pkgver}
	npm install
	npm run html
	npm run build
	bsdtar -xf web-ext-artifacts/libredirect-${pkgver}.zip -C ../libredirect.chromium
	cd ../libredirect.chromium
	jq --ascii-output --arg key "${pub_key}" '. + {key: $key}' manifest.json > manifest.json.new
	mv manifest.json.new manifest.json

	# Pack libredirect into the CRX format.
	cd ..
	chromium --user-data-dir="$(mktemp -d chromium-pack-XXXXXX)" --pack-extension=libredirect.chromium --pack-extension-key=private_key.pem
	mv libredirect.chromium.crx "${pkgname}-${pkgver}.crx"
}

package() {
    install -Dm644 -t "${pkgdir}/usr/share/chromium/extensions/" ${ext_identifier}.json
    install -Dm644 -t "${pkgdir}/usr/lib/${pkgname}/" "${pkgname}-${pkgver}.crx"
}
