python制作内部使用包 
===========================

devpi
----------------
先登录 devpi login user --password=pwd, 然后上传 devpi upload
需要当前目录有setup.py文件::

		import os

		from setuptools import find_packages, setup

		with open(os.path.join(os.path.dirname(__file__), 'README.rst')) as readme:
		    README = readme.read()

		# allow setup.py to be run from any path
		os.chdir(os.path.normpath(os.path.join(os.path.abspath(__file__), os.pardir)))

		setup(
		    name='pkg_name',
		    version='1.5',
		    packages=find_packages(),
		    include_package_data=True,
		    license='Med Data Quest Inc',
		    description='package description',
		    long_description=README,
		    url='http://example.com',
		    author='hstk',
		    author_email='hstk30@icloud.com',
		    classifiers=[
		        'Environment :: Web Environment',
		        'Framework :: Django',
		        'Framework :: Django :: 1.11',
		        'Intended Audience :: Developers',
		        'License :: Med Data Quest Inc (2011 - 2018)',
		        'Operating System :: OS Independent',
		        'Programming Language :: Python',
		        'Programming Language :: Python :: 2.7',
		        'Topic :: Internet :: WWW/HTTP',
		        'Topic :: Internet :: WWW/HTTP :: Dynamic Content',
		    ],
		)


pypi-server
-------------------



