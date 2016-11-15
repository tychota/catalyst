CONTEXT_FILES = []
BIG_PR_THRESHOLD = 500

SRC_REGEX = /^(src\/.*|native)\.js$/ # tolerant regex

JS_TEST_REGEX = /^.*\.test.js$/ # tolerant regex

VENDOR_DIR_REGEX = /^\/?src\/vendor\/.*$/

js_modified_files = git.modified_files.grep(SRC_REGEX)
vendor_modified_files = git.modified_files.grep(VENDOR_DIR_REGEX)

# `modified_files` include also `deleted_files` and `added_files`
has_app_changes = !(js_modified_files.empty?)
has_test_changes = !(git.modified_files.grep(JS_TEST_REGEX).empty?)

is_version_bump = begin
  diff = git.diff_for_file("package.json")
  if diff && diff.patch =~ /.*([+]|[-])\s{2}"version"[:].*[,]/
    true
  else
    false
  end
end

message(":tada: Version BUMP UP!") if is_version_bump

warn(":exclamation: Big PR") if git.lines_of_code > BIG_PR_THRESHOLD

unless github.api.organization_member?('styled-components', github.pr_author)
  if git.modified_files.include?("package.json")
    warn ":exclamation: External contributor has edited package.json"
  end
end

if github.pr_body.length < 5
  fail "Please provide a summary in the Pull Request description"
end

if !git.modified_files.include?("CHANGELOG.md") && has_app_changes
  fail("Please include a CHANGELOG entry. \nYou can find it at [CHANGELOG.md](https://github.com/styled-components/styled-components/blob/master/CHANGELOG.md).")
end

if !((js_modified_files - vendor_modified_files).empty?) && !has_test_changes
  warn("There're library changes, but not tests. That's OK as long as you're refactoring existing code.", sticky: false)
end


# Warn that changes to ThemeProvider.js, StyledComponent.js or StyledNativeComponent.js might
# be semver major changes if they touch the context stuff
#CONTEXT_FILES.each do |file|
#  file_regex = /\/#{file}/
#  warn("changes to #{file} might be semver major changes") unless js_modified_files.grep(file_regex).empty?
#end


# Don't let testing shortcuts be merged into master (fit, fdescribe, it.only, describe.only)
git.modified_files.grep(JS_TEST_REGEX).each do |file|
  next unless File.file?(file)
  contents = File.read(file)
  if file.end_with?("test.js")
    fail("`fit` left in tests (#{file})") if contents =~ /^\s*fit[(].*$/
    fail("`fdescribe` left in tests (#{file})") if contents =~ /^\s*fdescribe[(].*$/
    fail("`only` left in tests (#{file})") if contents =~ /^\s*(it|describe)[.]only[(].*$/
  end
end
