require 'tmpdir'
require 'fileutils'

def root_path
  File.expand_path('../.', __FILE__)
end

def commit
  @commit ||= (
    ENV['TRAVIS_COMMIT'] || `git rev-parse --abbrev-ref HEAD`.chomp
  )
end

task :default do
  Dir.mktmpdir('gvm-test') do |tmpdir|
    begin
      _system_rslt = system(<<-EOSH)
        bash -c '
          #{root_path}/binscripts/gvm-installer #{commit} #{tmpdir}
          source #{tmpdir}/gvm/scripts/gvm
          builtin cd #{tmpdir}/gvm/tests
          tf --text *_comment_test.sh
        '
      EOSH
      raise(SystemCallError, "something bad happened") unless _system_rslt
    rescue SystemCallError => e
      raise e
    ensure
      # copy build logs to source directory
      FileUtils.mkdir("#{root_path}/build_logs") unless Dir.exist?("#{root_path}/build_logs")
      printf "Log files... (tmpdir: #{tmpdir})\n"
      Dir.glob("#{tmpdir}/**/*.log").each { |f| printf("%s\n", f) }
      FileUtils.cp(Dir.glob("#{tmpdir}/gvm/logs/*.log"), "#{root_path}/build_logs")
    end
  end
end

task :scenario do
  Dir["#{root_path}/tests/scenario/*_comment_test.sh"].each do |test|
    name = File.basename(test)
    puts "Running scenario #{name}..."
    Dir.mktmpdir('gvm-test') do |tmpdir|
      begin
        _system_rslt = system(<<-EOSH)
          bash -c '
            #{root_path}/binscripts/gvm-installer #{commit} #{tmpdir}
            source #{tmpdir}/gvm/scripts/gvm
            builtin cd #{tmpdir}/gvm/tests/scenario
            tf --text #{name}
          '
        EOSH
        raise(SystemCallError, "something bad happened") unless _system_rslt
      rescue SystemCallError => e
        raise e
      ensure
        # copy build logs to source directory
        FileUtils.mkdir("#{root_path}/build_logs") unless Dir.exist?("#{root_path}/build_logs")
        printf "Log files... (tmpdir: #{tmpdir})\n"
        Dir.glob("#{tmpdir}/**/*.log").each { |f| printf("%s\n", f) }
        FileUtils.cp(Dir.glob("#{tmpdir}/gvm/logs/*.log"), "#{root_path}/build_logs")
      end
    end
  end
end

