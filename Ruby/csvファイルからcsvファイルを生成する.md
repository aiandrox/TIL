```rb
require 'csv'

class Csv
  def initialize(csv_file)
    @export_rows = []
    @csv_file = csv_file
  end

  def export_csv_file
    # push_export_rowで一行ずつrowを挿入していることが前提
    CSV.generate(encoding: Encoding::CP932) do |csv|
      csv << imported_csv_data_with_headers.headers
      export_rows.each do |row|
        csv << row
      end
    end
  end

  def push_export_row(row)
    # rowを挿入する
    row = row_hash.values if row.is_a?(Hash)
    export_rows << row
  end

  def imported_csv_data_with_headers
    @imported_csv_data ||= CSV.open(csv_file.path,
                                    headers: true,
                                    encoding: Encoding::CP932)
  end

  private

  attr_reader :csv_file, :export_rows

  def exporting_file_name
    @exporting_file_name ||= csv_file.original_filename
                                     .split('.csv')
                                     .push("_#{Time.zone.now.strftime('%Y%m%d%H%M%S')}")
                                     .join + '.csv'
  end
end
```

