task :check, [:file] do |task, args|
  exceptions = %w(
    https://github.com/chrisseaton/cspassword
    https://labs.oracle.com/
    https://www.linkedin.com/
    https://github.com/dmlloyd/openjdk.git
    https://www.youtube.com/watch?v=Hqw57GJSrac
    https://github.com/graalvm/mx.git
    https://github.com/graalvm/graal.git
    https://dl.acm.org/
    https://github.com/chrisseaton/graalvm-ten-things.git
    https://localhost:8080
    https://localhost:3000
  )

  sh %(
    bundle exec awesome_bot \
      --skip-save-results --allow-dupe \
      --white-list #{exceptions.join(',')} \
      #{args.file || '`find . -name \'*.md\'`'}
  )
end

task :build do
  sh 'bundle exec jekyll build'
end
