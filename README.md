[![Build Status](https://travis-ci.org/skaji/Parallel-Pipes.svg?branch=master)](https://travis-ci.org/skaji/Parallel-Pipes)

# NAME

Parallel::Pipes - parallel processing using pipe(2) for communication and synchronization

# SYNOPSIS

    use Parallel::Pipes;

    my $pipes = Parallel::Pipes->new(5, sub {
      # this is a worker code
      my $task = shift;
      my $result = do_work($task);
      return $result;
    });

    my $queue = Your::TaskQueue->new;
    # wrap Your::TaskQueue->get
    my $get; $get = sub {
      my $queue = shift;
      if (my @task = $queue->get) {
        return @task;
      }
      if (my @written = $pipes->is_written) {
        my @ready = $pipes->is_ready(@written);
        $queue->register($_->read) for @ready;
        return $queue->$get;
      } else {
        return;
      }
    };

    while (my @task = $queue->$get) {
      my @ready = $pipes->is_ready;
      $queue->register($_->read) for grep $_->is_written, @ready;
      my $min = List::Util::min($#task, $#ready);
      for my $i (0..$min) {
        # write tasks to pipes which are ready
        $ready[$i]->write($task[$i]);
      }
    }

    $pipes->close;

# DESCRIPTION

**THIS IS EXPERIMENTAL**.

Parallel processing is essential, but it is also difficult:

- How can we synchronize our workers?

    More precisely, how to detect our workers are ready or finished.

- How can we communicate with our workers?

    More precisely, how to collect results of tasks.

Parallel::Pipes tries to solve these problems with `pipe(2)` and `select(2)`.

[App::cpm](https://metacpan.org/pod/App::cpm), a fast CPAN module installer, uses Parallel::Pipes.
Please look at [App::cpm](https://github.com/skaji/cpm/blob/master/lib/App/cpm.pm)
or [eg directory](https://github.com/skaji/Parallel-Pipes/tree/master/eg) for real world usages.

<div>
    <a href="https://raw.githubusercontent.com/skaji/Parallel-Pipes/master/author/image.png"><img src="https://raw.githubusercontent.com/skaji/Parallel-Pipes/master/author/image.png" alt="image" class="img-responsive"></a>
</div>

# METHOD

## new

    my $pipes = Parallel::Pipes->new($number, $code);

The constructor, which takes

- number

    The number of workers.

- code

    Worker's code.

## is\_ready

    my @ready = $pipes->is_ready;
    my @ready = $pipes->is_ready(@pipes);

Get pipes which are ready to write.

## is\_written

    my @written = $pipes->is_written;

Get pipes which are written.

## close

    $pipes->close;

Close pipes (also shutdown workers).

# AUTHOR

Shoichi Kaji <skaji@cpan.org>

# COPYRIGHT AND LICENSE

Copyright 2016 Shoichi Kaji <skaji@cpan.org>

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.
