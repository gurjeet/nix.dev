.. _github-actions:

.. meta::
  :description: Continuous Integration with GitHub Actions and Cachix
  :keywords: CI, Continuous Integration, GitHub Actions, Cachix, Binary Cache, Nix

Continuous Integration with GitHub Actions
==========================================

In this tutorial, we'll show you **a few short steps** to get started using `GitHub Actions <https://github.com/features/actions>`_ as your continuous integration (CI) workflow for commits and pull requests.

Caching builds using Cachix
---------------------------

One benefit of Nix is that **CI can build and cache developer environments for every project** on every branch using binary caches.

An important aspect of CI is the feedback loop of, **how many minutes does the build take to finish?**

Using `Cachix <https://cachix.org/>`_ you'll never have to waste time building a derivation twice, and you'll share built derivations with all your developers.

After each job, just-built derivations are pushed to your binary cache.

Before each job, derivations to be built are first substituted (if they exist) from your binary cache.


1. Creating your first binary cache
***********************************

It's recommended to have different binary caches per team, depending who will have write/read access to it.

Fill out the form on the `create binary cache <https://app.cachix.org/cache>`_ page. 

On your freshly created binary cache, follow the **Push binaries** tab instructions.


2. Setting up secrets
*********************

On your GitHub repository or organization (for use across all repositories):

1. Click on ``Settings``. 
2. Click on ``Secrets``.
3. Add your previously generated secrets (``CACHIX_SIGNING_KEY`` and/or ``CACHIX_AUTH_TOKEN``).


Setting up GitHub Actions
-------------------------

Create ``.github/workflows/test.yml`` with:

.. code:: yaml

    name: "Test"
    on:
      pull_request:
      push:
    jobs:
      tests:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v2.4.0
        - uses: cachix/install-nix-action@v15
          with:
            nix_path: nixpkgs=channel:nixos-unstable
        - uses: cachix/cachix-action@v10
          with:
            name: mycache
            # If you chose signing key for write access
            signingKey: '${{ secrets.CACHIX_SIGNING_KEY }}'
            # If you chose API tokens for write access OR if you have a private cache
            authToken: '${{ secrets.CACHIX_AUTH_TOKEN }}'
        - run: nix-build
        - run: nix-shell --run "echo OK"

Once you commit and push to your GitHub repository,
you should see status checks appearing on commits and PRs.


Next steps
----------

- See `GitHub Actions workflow syntax <https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions>`_

- To quickly setup a Nix project read through 
  `Getting started Nix template <https://github.com/nix-dot-dev/getting-started-nix-template>`_.
