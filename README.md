# computed_field_report_pivot



    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        # overrides the default read_group in order to compute the computed fields manually for the group
        fields_list = {'duration'}
        fields = {field.split(':', 1)[0] if field.split(':', 1)[0] in fields_list else field for field in fields}
        result = super(Request, self).read_group(domain, fields, groupby, offset=offset, limit=limit,
                                                                orderby=orderby, lazy=lazy)
        if any(x in fields for x in fields_list):
            for group_line in result:

                # initialise fields to compute to 0 if they are requested
                if 'duration' in fields:
                    group_line['duration'] = 0

                if group_line.get('__domain'):
                    all_budget_lines_that_compose_group = self.search(group_line['__domain'])
                else:
                    all_budget_lines_that_compose_group = self.search([])
                for budget_line_of_group in all_budget_lines_that_compose_group:
                    if 'duration' in fields:
                        group_line['duration'] += budget_line_of_group.duration
        return result
