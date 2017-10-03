# Buckaroo
# https://buckaroo.pm
# TODO: Make a true port that does not use autotools

from os import path
from hashlib import sha256

def merge_dicts(x, y):
  z = x.copy()
  z.update(y)
  return z

genrule(
  name = 'configure',
  out = 'out',
  srcs = glob([
    '*.ac',
    '*.sh',
    '*.in',
    'bin/**/*.in',
    'build-aux/**/*',
    'doc/**/*.in',
    'doc/**/*.xsl',
    'm4/**/*.m4',
    'include/**/*.in',
    'include/**/*.sh',
    'test/**/*.in',
  ]),
  cmd = 'cp -r $SRCDIR $OUT && cd $OUT && ./autogen.sh && ./configure',
)

def extract(rule, subpath):
  basename = path.basename(subpath)
  filename, extension = path.splitext(basename)
  name = filename + '-' + sha256(rule + subpath).hexdigest()[:16]
  genrule(
    name = name,
    out = filename + extension,
    cmd = 'cp $(location ' + rule + ')/' + subpath + ' $OUT',
  )
  return ':' + name

jemalloc_h = extract(':configure', 'include/jemalloc/jemalloc.h')

cxx_library(
  name = 'jemalloc',
  header_namespace = '',
  exported_headers = merge_dicts(subdir_glob([
    ('include', 'jemalloc/*.h'),
  ]), {
    'jemalloc/jemalloc.h': jemalloc_h,
  }),
  headers = merge_dicts(subdir_glob([
    ('include', 'jemalloc/*.h'),
    ('include', 'jemalloc/internal/**/*.h'),
  ]), {
    'jemalloc/jemalloc.h': jemalloc_h,
    'jemalloc/internal/jemalloc_preamble.h':
      extract(':configure', 'include/jemalloc/internal/jemalloc_preamble.h'),
    'jemalloc/internal/jemalloc_internal_defs.h':
      extract(':configure', 'include/jemalloc/internal/jemalloc_internal_defs.h'),
    'jemalloc/internal/size_classes.h':
      extract(':configure', 'include/jemalloc/internal/size_classes.h'),
  }),
  srcs = glob([
    'src/**/*.c',
  ]),
  preprocessor_flags = [
    '-DJEMALLOC_NO_PRIVATE_NAMESPACE=1',
  ],
  visibility = [
    'PUBLIC',
  ],
)
