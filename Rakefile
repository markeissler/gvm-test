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
      system(<<-EOSH) || raise(SystemCallError, "system shell (bash) call failed")
        bash -c '
          #{root_path}/binscripts/gvm-installer #{commit} #{tmpdir}
        ' || exit 1
        EOSH
      system(<<-EOSH) || raise(SystemCallError, "system shell (bash) call failed")
        bash -c '
          source #{tmpdir}/gvm/scripts/gvm
          builtin cd #{tmpdir}/gvm/tests
          tf --text *_comment_test.sh
        '
      EOSH
    rescue SystemCallError => e
      raise e
    ensure
      # copy build logs to source directory
      FileUtils.mkdir("#{root_path}/build_logs") unless Dir.exist?("#{root_path}/build_logs")
      printf "Log files (1)... (tmpdir: #{tmpdir})\n"
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
        system(<<-EOSH) || raise(SystemCallError, "system shell (bash) call failed")
          bash -c '
            #{root_path}/binscripts/gvm-installer #{commit} #{tmpdir}
            source #{tmpdir}/gvm/scripts/gvm
            builtin cd #{tmpdir}/gvm/tests/scenario
            tf --text #{name}
          '
        EOSH
      rescue SystemCallError => e
        raise e
      ensure
        # copy build logs to source directory
        FileUtils.mkdir("#{root_path}/build_logs") unless Dir.exist?("#{root_path}/build_logs")
        printf "Log files (2)... (tmpdir: #{tmpdir})\n"
        Dir.glob("#{tmpdir}/**/*.log").each { |f| printf("%s\n", f) }
        FileUtils.cp(Dir.glob("#{tmpdir}/gvm/logs/*.log"), "#{root_path}/build_logs")
      end
    end
  end
end

