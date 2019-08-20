# Installation

## Clone OQS openssl

	git clone  https://github.com/dimisik/openssl.git
	cd openssl
	git checkout add_falcon_tls_1_3

## Clone, Build, Install liboqs

	cd ..
	git clone https://github.com/dimisik/liboqs.git 
	cd liboqs
	git checkout add_falcon
	autoreconf -i
	./configure --prefix=<PATH to openssl>/openssl/oqs --enable-shared=no
	make -j
	sudo make install
 
## Build, Install openssl

	cd ../openssl
	./config no-shared --prefix=<PATH>/oqs_openssl --openssldir=<PATH>/oqs_openssl
	sudo make install

## Build, Install liboqs in oqs_openssl

	cd ../liboqs
	autoreconf -i
	./configure --prefix=<PATH to oqs_openssl>/oqs_openssl/oqs --enable-shared=no
	make -j
	sudo make install

## Download and Build nginx and dependences
 
	sudo apt-get install libpcre3 libpcre3-dev
	wget http://nginx.org/download/nginx-1.17.0.tar.gz
	tar zxvf nginx-1.17.0.tar.gz
	cd nginx-1.17.0
	./configure --prefix=<PATH>/nginx --with-http_ssl_module --with-openssl=<PATH to oqs_openssl>/oqs_openssl --without-http_gzip_module

Changes in objs/Makefile:
		- Remove -I <PATH to oqs_openssl>/oqs_openssl/.openssl/include
		  and add -I /usr/local/oqs_openssl/include \
	              -I /usr/local/oqs_openssl/oqs/include \
		  in ALL_INCS =

        - Replace <PATH to oqs_openssl>/oqs_openssl/.openssl/include/openssl/ssl.h \
		    with <PATH to oqs_openssl>/oqs_openssl/include/openssl/ssl.h \
			in CORE_DEPS =
			
		- Add <PATH to oqs_openssl>/oqs_openssl/oqs/include/oqs/oqs.h \
			in CORE_DEPS =	
	
        - Replace -I <PATH to oqs_openssl>/oqs_openssl/.openssl/include \ 
		    with -I <PATH to oqs_openssl>/oqs_openssl/include \
			in CORE_INCS =
		
		- Add  -I <PATH to oqs_openssl>/oqs_openssl/oqs/include \
			in CORE_INCS =	

		- Replace <PATH to oqs_openssl>/oqs_openssl/.openssl/lib/libssl.a <PATH to oqs_openssl>/oqs_openssl/.openssl/lib/libcrypto.a
		with <PATH to oqs_openssl>/oqs_openssl/lib/libssl.a <PATH to oqs_openssl>/oqs_openssl/lib/libcrypto.a
		
	    - Add <PATH to oqs_openssl>/oqs_openssl/oqs/lib/liboqs.a  

		
		- Comment openssl rebuid 
		#<PATH to oqs_openssl>/oqs_openssl/.openssl/include/openssl/ssl.h:	objs/Makefile
		#	cd <PATH to oqs_openssl>/oqs_openssl \
		#	&& if [ -f Makefile ]; then $(MAKE) clean; fi \
		#	&& ./config --prefix=<PATH to oqs_openssl>/oqs_openssl/.openssl no-shared no-threads  \
		#	&& $(MAKE) \
		#	&& $(MAKE) install_sw LIBDIR=lib
		
Changes in src/event/ngx_event_openssl.c
		-(For compatibility with openssl 1.1.x) EVP_MD_CTX_create() and EVP_MD_CTX_destroy() were renamed to EVP_MD_CTX_new() and EVP_MD_CTX_free() in OpenSSL 1.1.
		- Rename EVP_MD_CTX_create() and EVP_MD_CTX_destroy() to EVP_MD_CTX_new() and EVP_MD_CTX_free() respectively

	make
	sudo make install


## Download and Build siege

	git clone https://github.com/JoeDog/siege.git
	cd siege 
	utils/bootstrap 
	./configure --prefix=<PATH>/oqs_siege --with-ssl=<PATH to oqs_openssl>

Changes in siege/Makefile:
		- SSL_LIBS = -lssl -lcrypto -loqs -ldl
	
		- Add I<PATH to oqs_openssl>/oqs_openssl/oqs/include
			in SSL_INCLUDE  
		- Add L<PATH to oqs_openssl>/oqs_openssl/oqs/lib
			in SSL_LDFLAGS
	
Changes in siege/src/Makefile:
		- SSL_LIBS = -lssl -lcrypto -loqs -ldl
	
		- Add I<PATH to oqs_openssl>/oqs_openssl/oqs/include
			in SSL_INCLUDE  
		- Add L<PATH to oqs_openssl>/oqs_openssl/oqs/lib
			in SSL_LDFLAGS	
	
		- Remove $(PTHREAD_LDFLAGS) from 
				AM_LDFLAGS = $(SSL_LDFLAGS) $(Z_LDFLAGS) $(PTHREAD_LDFLAGS)
		  and add it in LIBS so that
				LIBS = $(SSL_LIBS) $(Z_LIBS) $(PTHREAD_LDFLAGS)
				
		- Add <PATH to liboqs directory>/src/sig/sig.$(OBJEXT)
		     in am_siege_OBJECTS


	make
	make uninstall (if you have an older version installed in PREFIX)
	make install

Example siege 
	/opt/siege/bin/siege -b -t1S  https://cisco.com

