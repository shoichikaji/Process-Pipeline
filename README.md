[![Build Status](https://travis-ci.org/skaji/Process-Pipeline.svg?branch=master)](https://travis-ci.org/skaji/Process-Pipeline)

# NAME

Process::Pipeline - execute processes as pipeline

# SYNOPSIS

In shell:

    $ zcat access.log.gz | grep -v 127.0.0.1 | grep -c POST

In perl5:

    use Process::Pipeline;

    my $pipeline = Process::Pipeline->new
      ->push(sub { my $p = shift; $p->cmd("zcat", "access.log.gz")   })
      ->push(sub { my $p = shift; $p->cmd("grep", "-v", "127.0.0.1") })
      ->push(sub { my $p = shift; $p->cmd("grep", "-c", "POST"       });

    my $r = $pipeline->start;
    if ($r->is_success) {
       my $fh = $result->fh; # output filehandle of $pipeline
       say <$fh>;
    }

In perl5 with DSL:

    use Process::Pipeline::DSL;

    my $pipeline = proc { "zcat", "access.log.gz"   }
                   proc { "grep", "-v", "127.0.0.1" }
                   proc { "grep", "-c", "POST"      };

    my $r = $pipeline->start;
    if ($r->is_success) {
       my $fh = $result->fh; # output filehandle of $pipeline
       say <$fh>;
    }

# DESCRIPTION

Process::Pipeline helps you write a pipeline of processes.

# MOTIVATION

It is known that we should avoid shell-invocation in perl.
But, because the notation of shell is very convenient,
I sometimes find myself invoking shell. Oops.

The main reason for invoking shell in perl is
that perl does not have as convenient notation as shell has.

Process::Pipeline try to give an easy pipeline notation to perl.

Why don't you change

    chomp(my $num = `zcat access.log.gz | grep -v 127.0.0.1 | grep -c POST`);

into

    use Process::Pipeline::DSL;
    my $p = proc { "zcat", "access.log.gz"   }
            proc { "grep", "-v", "127.0.0.1" }
            proc { "grep", "-c", "POST"      };
    my $r = $p->start;
    if ($r->is_success) {
      my $fh = $r->fh;
      chomp(my $num = <$fh>);
    }

# METHODS

## new

    my $pipeline = Process::Pipeline->new;

Constructor.

## push

    $pipeline->push(sub {
      my $p = shift;
      $p->cmd("zcat", "access.log.gz");
    });
    $pipeline->push(sub {
      my $p = shift;
      $p->set("2>", "/dev/null");
      $p->cmd("zcat", "access.log.gz");
    });

Push a Process::Pipeline::Process object to the pipeline.

### start

    my $result = $pipeline->start;

Start the pipeline. It returns a Process::Pipeline::Result object.

    my $result = $pipeline->start;
    my $bool   = $reuslt->is_success; # all commands exit successfully
    my $fh     = $reuslt->fh;         # pipeline's output filehandle

# DSL

There is a DSL for Process::Pipeline. Process::Pipeline::DSL exports
`proc` and `set` functions, and you can construct pipelines easily.

    use Process::Pipeline::DSL;
    my $p = proc { "git", "archive", "--format=tar", "--prefix=repo/", "HEAD" }
            proc { set ">" => "repo.tar.gz"; "gzip" };
    my $r = $p->start;

# COPYRIGHT AND LICENSE

Copyright 2016 Shoichi Kaji <skaji@cpan.org>

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.
