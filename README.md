const columns = [
    {
      accessorFn: (row: Standard) => row.display_stdNo,
      id: "display_stdNo",
      meta: { headerClassName: "bg-gray-500 text-white" },
      header: ({ column }: { column: any }) => (
        <DataGridColumnHeader title="Standard No." column={column} />
      ),
      cell: ({ row }: { row: { original: Standard } }) => (
        <span>{row.original.display_stdNo}</span>
      ),
    },
    {
      accessorFn: (row: Standard) => row.stdPrices,
      id: "stdPrices",
      meta: { headerClassName: "bg-gray-500 text-white" },
      header: ({ column }: { column: any }) => (
        <DataGridColumnHeader title="Unit Rate" column={column} />
      ),
      cell: ({ row }: { row: { original: Standard } }) => (
        <div>
          {row.original.stdPrices?.length > 0 ? (
            row.original.stdPrices.map((priceList, index) => (
              <div key={index} className="inline-block mr-4">
                <p className="inline">
                  {priceList.price}
                  {priceList.formate_id === 1
                    ? " PDF"
                    : priceList.formate_id === 2
                      ? " PRINT"
                      : " Unknown"}
                </p>
              </div>
            ))
          ) : (
            <span>N/A</span>
          )}
        </div>
      ),
    },
    {
      accessorFn: (row: Standard) => row.priceInINR,
      id: "priceInINR",
      meta: { headerClassName: "bg-gray-500 text-white" },
      header: ({ column }: { column: any }) => (
        <DataGridColumnHeader title="Price (INR)" column={column} />
      ),
      cell: ({ row }: { row: { original: Standard } }) => {
        const curr = currencyData.find((c) => c.id === 4);
        const price =
          row.original.stdPrices?.length > 0
            ? row.original.stdPrices[0].price * (curr?.currency_amount || 1)
            : "N/A";
        return <span>{price}</span>;
      },
    },
    {
      accessorFn: (row: Standard) => row.title,
      id: "title",
      meta: { headerClassName: "bg-gray-500 text-white" },
      header: ({ column }: { column: any }) => (
        <DataGridColumnHeader title="Title" column={column} />
      ),
      cell: ({ row }: { row: { original: Standard } }) => (
        <span>{row.original.title}</span>
      ),
    },
    {
      id: "actions",
      meta: { headerClassName: "bg-gray-500 text-white" },
      cell: ({ row }: { row: { original: Standard } }) => (
        <button className="bg-blue-500 rounded hover:bg-gray-100 items-center p-2">
          <PanelRightOpen size={16} strokeWidth={3} />
        </button>
      ),
    },
  ];
